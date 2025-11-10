ll




// AESEncryption.kt
package com.hillrom.servicetool.service.dynamo

import java.io.*
import javax.crypto.Cipher
import javax.crypto.CipherInputStream
import javax.crypto.spec.IvParameterSpec
import javax.crypto.spec.SecretKeySpec

object AESEncryption {

    // If the BAS header doesn’t provide a key, you can fall back to this.
    // Replace with your real default or remove the fallback entirely.
    private const val DEFAULT_KEY_HEX = "00112233445566778899AABBCCDDEEFF"

    fun keyFromHex(hex: String): ByteArray = hex
        .trim()
        .replace(" ", "")
        .chunked(2)
        .map { it.toInt(16).toByte() }
        .toByteArray()

    fun String.hexToBytes(): ByteArray = this
        .trim()
        .replace(" ", "")
        .chunked(2)
        .map { it.toInt(16).toByte() }
        .toByteArray()

    fun ByteArray.bytesToHex(): String = joinToString("") { "%02X".format(it) }

    private data class EncParams(val key: ByteArray, val iv: ByteArray)

    /**
     * Looks for header lines like:
     *   KEY=001122... (hex)
     *   IV =A1B2C3... (hex, 16 bytes -> 32 hex chars)
     * near the beginning of the .bas.
     * Return null if not present.
     */
    fun readEncParamsFromBasHeader(bas: File): EncParams? {
        bas.inputStream().buffered().use { ins ->
            ins.reader(Charsets.UTF_8).buffered().use { br ->
                repeat(64) {   // scan a few header lines; adjust if needed
                    val line = br.readLine() ?: return@repeat
                    if (line.startsWith("KEY=", ignoreCase = true)) {
                        val keyHex = line.substringAfter("KEY=").trim()
                        val ivLine = br.readLine() ?: return null
                        val ivHex = ivLine.substringAfter("IV=", "").trim()
                        return EncParams(
                            key = if (keyHex.isNotEmpty()) keyHex.hexToBytes()
                                  else keyFromHex(DEFAULT_KEY_HEX),
                            iv  = ivHex.hexToBytes()
                        )
                    }
                }
            }
        }
        return null
    }

    /**
     * Simple AES-CBC/PKCS5 decrypt (blocking) from input file to output file.
     * Switch algorithm/mode/padding here if your producer uses something else.
     */
    fun decryptFileBlocking(
        input: File,
        output: File,
        keyBytes: ByteArray,
        iv: ByteArray,
        algorithm: String = "AES/CBC/PKCS5Padding"
    ): Boolean {
        require(iv.size == 16) { "IV must be 16 bytes" }

        return try {
            val secret = SecretKeySpec(keyBytes, "AES")
            val cipher = Cipher.getInstance(algorithm)
            cipher.init(Cipher.DECRYPT_MODE, secret, IvParameterSpec(iv))

            input.inputStream().use { fis ->
                output.outputStream().use { fos ->
                    CipherInputStream(fis, cipher).use { cis ->
                        cis.copyTo(fos)
                    }
                }
            }
            true
        } catch (e: Exception) {
            // keep logs optional/quiet if you prefer
            false
        }
    }
}




// Treat any *.enc as encrypted
private fun File.isEnc(): Boolean =
    name.endsWith(".enc", ignoreCase = true)

/** Decrypt *.enc next to itself (same name without .enc) and return the decrypted file. */
private fun File.decryptedIfNeededBlocking(basFile: File): File {
    if (!isEnc()) return this

    val out = File(parentFile, name.removeSuffix(".enc")) // expected *.tar.gz
    if (out.exists()) out.delete()

    // Pull KEY/IV from the BAS header; adapt if you store them elsewhere.
    val params = AESEncryption.readEncParamsFromBasHeader(basFile)
        ?: throw IllegalArgumentException("BAS header missing KEY/IV for $name")

    val ok = AESEncryption.decryptFileBlocking(
        input   = this,
        output  = out,
        keyBytes= params.key,
        iv      = params.iv
    )
    if (!ok) throw IllegalArgumentException("Decryption failed for $name")
    return out
}




override suspend fun extractFirmwareFile(
    firmwareFileInputStream: InputStream
): Outcome<SoftwareReleaseManifest> {
    val basFile = File("${context.filesDir.absolutePath}/firmware.bas")
    val os = basFile.outputStream()
    return try {
        firmwareFileInputStream.copyTo(os)
        os.flush()

        // Clean the extraction folder
        extractionFolder.listFiles()?.forEach { it.deleteRecursively() }
        extractionFolder.mkdirs()

        // Your existing extraction (zip/tar handling happens inside this)
        if (!dynamo.extractBasFile(basFile.absolutePath, extractionFolder.absolutePath)) {
            ProLog.w(MODULE_NAME, "Unable to extract bas file: ${basFile.name}")
            return Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)
        }

        // NEW: auto-decrypt any *.enc payloads right after extraction
        extractionFolder.listFiles()?.forEach { f ->
            if (f.isFile && f.isEnc()) {
                try {
                    f.decryptedIfNeededBlocking(basFile)    // creates sibling without .enc
                } catch (e: Exception) {
                    ProLog.e(MODULE_NAME, "decrypt failed: ${f.name}")
                    return Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)
                }
            }
        }

        // Find/parse manifest (now plain text after decrypt)
        val manifestFile = extractionFolder.listFiles()
            ?.firstOrNull { it.name.contains("manifest", ignoreCase = true) }
            ?: return Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)

        val releaseManifest = try {
            manifestFile.inputStream().use { ManifestParser().parse(it) }
        } catch (e: Exception) {
            ProLog.w(MODULE_NAME, "Unable to parse manifest file.")
            return Outcome.Error(e)
        }

        Outcome.Ok(releaseManifest)
    } finally {
        try { os.close() } catch (_: Throwable) {}
    }
}



override suspend fun evaluateFirmwareCompatibility(
    activeOSVersion: DynamoYoctoOS,
    releaseManifest: SoftwareReleaseManifest
): Outcome<DynamoFirmwareCompatibilityStatus> {

    var containsOS = false
    var osPkgFound = false
    var pkgOsVersion = DynamoYoctoOS.INVALID

    // Helper that gives you the actual file to test (plain or decrypted)
    fun resolveCandidate(name: String, basFile: File): File? {
        val direct = File(extractionFolder, name)
        if (direct.exists()) return direct
        val enc = File(extractionFolder, "$name.enc")
        return if (enc.exists()) enc.decryptedIfNeededBlocking(basFile) else null
    }

    val basFile = File("${context.filesDir.absolutePath}/firmware.bas") // same as extract step

    extractionFolder.listFiles()?.forEach { f ->
        // OS package (plain or enc)
        if (f.name == ospkgTarGzFileName || f.name == "${ospkgTarGzFileName}.enc") {
            osPkgFound = true
            val candidate = resolveCandidate(ospkgTarGzFileName, basFile)
                ?: return@forEach
            when (val ver = Dynamo.extractYoctoVersion(context, candidate)) {
                is Outcome.Ok -> {
                    lastEvaluatedFwUpdateOsVersion = SemVer.fromString(ver.value)
                    containsOS = shouldContainOS(lastEvaluatedFwUpdateOsVersion)
                    pkgOsVersion = DynamoYoctoOS.fromVersion(lastEvaluatedFwUpdateOsVersion)
                }
                is Outcome.Error -> {
                    ProLog.e(MODULE_NAME, "Could not find yocto version in ${candidate.name}. Result: ${ver.error}")
                    return Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)
                }
            }
            return@forEach
        }

        // DI package (plain or enc) — unchanged logic, just the same resolve
        if (f.name == dynamoOsTarGzFileName || f.name == "${dynamoOsTarGzFileName}.enc") {
            val candidate = resolveCandidate(dynamoOsTarGzFileName, basFile) ?: return@forEach
            when (val ver = Dynamo.extractYoctoVersion(context, candidate)) {
                is Outcome.Ok -> {
                    lastEvaluatedFwUpdateOsVersion = SemVer.fromString(ver.value)
                    containsOS = shouldContainOS(lastEvaluatedFwUpdateOsVersion)
                    pkgOsVersion = DynamoYoctoOS.fromVersion(lastEvaluatedFwUpdateOsVersion)
                }
                is Outcome.Error -> {
                    ProLog.e(MODULE_NAME, "Could not find yocto version in ${candidate.name}. Result: ${ver.error}")
                    return Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)
                }
            }
            return@forEach
        }
    }

    // Fallback: if OS wasn’t identified but app tar is present, keep your old default
    if (pkgOsVersion == DynamoYoctoOS.INVALID) {
        extractionFolder.listFiles()?.forEach {
            if (it.name == dynamoAppTar) {
                pkgOsVersion = DynamoYoctoOS.DUNFELL
                return@forEach
            }
        }
    }

    return Outcome.Ok(
        if (!osPkgFound) DynamoFirmwareCompatibilityStatus.OSUpdateNotIncluded
        else if (containsOS) DynamoFirmwareCompatibilityStatus.UpdateCompatible
        else DynamoFirmwareCompatibilityStatus.UpdateIncompatible
    )
}





