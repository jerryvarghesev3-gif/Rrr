ll



package com.hillrom.servicetool.service.dynamo

import java.io.File
import java.io.FileInputStream
import java.io.FileOutputStream
import java.security.SecureRandom
import javax.crypto.Cipher
import javax.crypto.CipherInputStream
import javax.crypto.spec.GCMParameterSpec
import javax.crypto.spec.IvParameterSpec
import javax.crypto.spec.SecretKeySpec

object AESEncryption {

    enum class Mode { CBC_PKCS5, GCM }

    // ---- small utils ----
    fun String.hexToBytes(): ByteArray {
        val s = trim().replace(" ", "")
        require(s.length % 2 == 0) { "hex length must be even" }
        return ByteArray(s.length / 2) { i ->
            ((s[2*i].digitToInt(16) shl 4) or (s[2*i+1].digitToInt(16))).toByte()
        }
    }

    data class EncParams(val key: ByteArray, val iv: ByteArray)

    /**
     * Looks for lines near the start of the BAS that contain:
     *   KEY=001122... (hex)
     *   IV =A1B2C3... (hex)
     * Returns null if not found (caller can fall back to defaults/policy).
     */
    fun readEncParamsFromBasHeader(bas: File): EncParams? {
        FileInputStream(bas).buffered().use { ins ->
            ins.reader(Charsets.UTF_8).buffered().use { br ->
                repeat(64) {       // only skim first ~64 lines
                    val line = br.readLine() ?: return@repeat
                    if (line.startsWith("KEY=", ignoreCase = true)) {
                        val keyHex = line.substringAfter("KEY=").trim()
                        val ivHex  = br.readLine()?.substringAfter("IV=")?.trim() ?: return null
                        return EncParams(keyHex.hexToBytes(), ivHex.hexToBytes())
                    }
                }
            }
        }
        return null
    }

    /**
     * Blocking file decrypt. Use CBC/PKCS5 by default (what you showed in logs).
     * For GCM, pass mode = Mode.GCM and a 12-byte IV (nonce).
     */
    fun decryptFileBlocking(
        input: File,
        output: File,
        keyBytes: ByteArray,
        iv: ByteArray,
        mode: Mode = Mode.CBC_PKCS5
    ): Boolean {
        val algo = when (mode) {
            Mode.CBC_PKCS5 -> "AES/CBC/PKCS5Padding"
            Mode.GCM       -> "AES/GCM/NoPadding"
        }

        val key   = SecretKeySpec(keyBytes, "AES")
        val cipher = Cipher.getInstance(algo)

        when (mode) {
            Mode.CBC_PKCS5 -> {
                require(iv.size == 16) { "CBC IV must be 16 bytes" }
                cipher.init(Cipher.DECRYPT_MODE, key, IvParameterSpec(iv))
            }
            Mode.GCM -> {
                // typical: 12-byte nonce, 128-bit tag
                require(iv.size == 12) { "GCM nonce must be 12 bytes" }
                cipher.init(Cipher.DECRYPT_MODE, key, GCMParameterSpec(128, iv))
            }
        }

        FileInputStream(input).use { fis ->
            CipherInputStream(fis, cipher).use { cis ->
                FileOutputStream(output).use { fos ->
                    cis.copyTo(fos)
                }
            }
        }
        return true
    }
}




import java.io.File

private fun File.isEnc(): Boolean =
    name.endsWith(".enc", ignoreCase = true)

/** Decrypts “this” (which is *.enc) next to itself (same name minus .enc) using KEY/IV from the BAS header. */
private fun File.decryptNextToSelfBlocking(basFile: File, mode: AESEncryption.Mode = AESEncryption.Mode.CBC_PKCS5): File {
    val out = File(parentFile, name.removeSuffix(".enc"))
    if (out.exists()) out.delete()

    val params = AESEncryption.readEncParamsFromBasHeader(basFile)
        ?: throw IllegalArgumentException("BAS header missing KEY/IV for ${name}")

    val ok = AESEncryption.decryptFileBlocking(
        input   = this,
        output  = out,
        keyBytes= params.key,
        iv      = params.iv,
        mode    = mode
    )
    if (!ok) throw IllegalStateException("Decrypt failed for ${name}")
    return out
}



// inside your DynamoFirmwareUpdateService (or equivalent)
override suspend fun extractFirmwareFile(
    firmwareFileInputStream: InputStream
): Outcome<SoftwareReleaseManifest> {

    val basFile = File("${context.filesDir.absolutePath}/firmware_raw.bas")
    val os = basFile.outputStream()
    try {
        firmwareFileInputStream.copyTo(os)
        os.flush()

        // Clean extraction folder
        extractionFolder.listFiles()?.forEach { it.deleteRecursively() }
        extractionFolder.mkdirs()

        // 1) Unpack BAS (unchanged)
        if (!dynamo.extractBasFile(basFile.absolutePath, extractionFolder.absolutePath)) {
            ProLog.w(MODULE_NAME, msg = "Unable to extract bas file: ${basFile.name}")
            return Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)
        }

        // 2) Decrypt any "*.enc" members in-place (-> same name without .enc)
        extractionFolder.listFiles()?.forEach { f ->
            if (f.isFile && f.isEnc()) {
                try {
                    f.decryptNextToSelfBlocking(basFile)   // CBC by default
                    // optionally delete the *.enc after success:
                    // f.delete()
                } catch (e: Throwable) {
                    ProLog.e(MODULE_NAME, "decrypt failed: ${f.name}", e)
                    return Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)
                }
            }
        }

        // 3) Find and parse manifest (already plain-text if encrypted before)
        val manifestFile = extractionFolder
            .listFiles()
            ?.firstOrNull { it.name.contains("manifest", ignoreCase = true) }
            ?: return Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)

        val releaseManifest = try {
            manifestFile.inputStream().use { ManifestParser().parse(it) }
        } catch (e: Exception) {
            ProLog.w(MODULE_NAME, msg = "Unable to parse manifest file.", tr = e)
            return Outcome.Error(e)
        }

        return Outcome.Ok(releaseManifest)
    } catch (e: Exception) {
        ProLog.e(MODULE_NAME, "extractFirmwareFile failed", e)   // <-- 3-arg overload fixes your “too many arguments” hint
        return Outcome.Error(e)
    } finally {
        try { os.close() } catch (_: Throwable) {}
    }
}




// In your Bloc/ViewModel, small wrapper to stay off main thread
override suspend fun evaluateFirmwareCompatibility(
    activeOSVersion: DynamoYoctoOS,
    firmwareFileInputStream: InputStream,
): Outcome<DynamoFirmwareCompatibilityStatus> {

    update { s -> s.copy(firmwareCompatibilityStatus = DynamoFirmwareCompatibilityStatus.Evaluating) }

    return withContext(Dispatchers.IO) {
        // 1) Extract + (if needed) decrypt + parse manifest
        val manifestOutcome = eFirmwareService.extractFirmwareFile(firmwareFileInputStream)
        when (manifestOutcome) {
            is Outcome.Error -> Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)
            is Outcome.Ok -> {
                val manifest = manifestOutcome.value
                // (optional caching for UI)
                update { s -> s.copy(releaseManifest = manifest) }

                // 2) continue with your existing compatibility decision logic
                eFirmwareService.evaluateFirmwareCompatibility(
                    bedStatusBloc.getSomOS(),
                    manifest
                )
            }
        }
    }
}
