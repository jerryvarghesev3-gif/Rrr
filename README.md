ll




package com.hillrom.servicetool.service.dynamo

import android.content.Context
import com.hillrom.servicetool.common.Outcome
import com.hillrom.servicetool.common.ProLog
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.withContext
import java.io.*
import java.nio.charset.Charset
import java.security.SecureRandom
import javax.crypto.Cipher
import javax.crypto.CipherInputStream
import javax.crypto.spec.GCMParameterSpec
import javax.crypto.spec.IvParameterSpec
import javax.crypto.spec.SecretKeySpec

private const val MODULE_NAME = "FIRMWARE"

// ---------- AES helper (includes your encrypt/decrypt string) ----------
object AESEncryption {

    enum class Mode { CBC_PKCS5, GCM }

    /** Change this or remove if you *must* read KEY from BAS only. */
    private const val DEFAULT_KEY_HEX = "00112233445566778899AABBCCDDEEFF"

    fun keyFromHex(hex: String): ByteArray {
        val c = hex.trim().replace("\\s".toRegex(), "")
        require(c.length % 2 == 0) { "hex length must be even" }
        val out = ByteArray(c.length / 2)
        var i = 0
        var j = 0
        while (i < c.length) {
            out[j++] = (((c[i].digitToInt(16) shl 4) or c[i + 1].digitToInt(16)).toByte())
            i += 2
        }
        return out
    }

    // ---- File decrypt (blocking, for background thread use) ----
    fun decryptFileBlocking(
        input: File,
        output: File,
        keyBytes: ByteArray,
        iv: ByteArray,
        mode: Mode = Mode.CBC_PKCS5
    ): Boolean {
        try {
            val secret = SecretKeySpec(keyBytes, "AES")
            val cipher: Cipher = when (mode) {
                Mode.CBC_PKCS5 -> {
                    require(iv.size == 16) { "IV must be 16 bytes for CBC" }
                    Cipher.getInstance("AES/CBC/PKCS5Padding").apply {
                        init(Cipher.DECRYPT_MODE, secret, IvParameterSpec(iv))
                    }
                }
                Mode.GCM -> {
                    // If your producer actually uses GCM, use this branch and pass a 12-byte nonce.
                    require(iv.size in 12..16) { "Nonce 12–16 bytes expected for GCM" }
                    Cipher.getInstance("AES/GCM/NoPadding").apply {
                        init(Cipher.DECRYPT_MODE, secret, GCMParameterSpec(128, iv))
                    }
                }
            }

            input.inputStream().use { fis ->
                CipherInputStream(fis, cipher).use { cis ->
                    output.outputStream().use { fos ->
                        val buf = ByteArray(64 * 1024)
                        while (true) {
                            val n = cis.read(buf)
                            if (n <= 0) break
                            fos.write(buf, 0, n)
                        }
                        fos.flush()
                    }
                }
            }
            return true
        } catch (e: Throwable) {
            ProLog.e(MODULE_NAME, "decryptFileBlocking failed: ${e.message}")
            return false
        }
    }

    // ---- Your string helpers (AES-GCM) from screenshot ----
    private const val GCM_TAG_LEN = 16
    private const val GCM_NONCE_LEN = 12
    private const val ALGO_GCM = "AES/GCM/NoPadding"

    // Provide your own real key source instead of DEFAULT_KEY_HEX if needed.
    private val encryptionKey by lazy { SecretKeySpec(keyFromHex(DEFAULT_KEY_HEX), "AES") }

    fun encryptString(value: String): String {
        return try {
            val iv = ByteArray(GCM_NONCE_LEN).also { SecureRandom().nextBytes(it) }
            val gcmSpec = GCMParameterSpec(GCM_TAG_LEN * 8, iv)
            val cipher = Cipher.getInstance(ALGO_GCM)
            cipher.init(Cipher.ENCRYPT_MODE, encryptionKey, gcmSpec)
            val cipherText = cipher.doFinal(value.toByteArray())
            (iv + cipherText).encodeToHex()
        } catch (e: Exception) {
            ProLog.e(MODULE_NAME, e.stackTraceToString())
            ""
        }
    }

    fun decryptString(value: String): String {
        return try {
            val raw = value.decodeHex()
            require(raw.size >= GCM_NONCE_LEN + GCM_TAG_LEN) { "bad GCM blob" }
            val iv = raw.copyOfRange(0, GCM_NONCE_LEN)
            val data = raw.copyOfRange(GCM_NONCE_LEN, raw.size)
            val gcmSpec = GCMParameterSpec(GCM_TAG_LEN * 8, iv)
            val cipher = Cipher.getInstance(ALGO_GCM)
            cipher.init(Cipher.DECRYPT_MODE, encryptionKey, gcmSpec)
            val plain = cipher.doFinal(data)
            String(plain)
        } catch (e: Exception) {
            ProLog.e(MODULE_NAME, e.stackTraceToString())
            ""
        }
    }

    // --- tiny hex helpers ---
    private fun ByteArray.encodeToHex(): String =
        joinToString("") { "%02X".format(it.toInt() and 0xFF) }

    private fun String.decodeHex(): ByteArray {
        val s = trim().replace("\\s".toRegex(), "")
        require(s.length % 2 == 0) { "hex must be even length" }
        return s.chunked(2).map { it.toInt(16).toByte() }.toByteArray()
    }
}

// ---------- Service ----------
open class DynamoFirmwareService(private val context: Context) : IDynamoFirmwareUpdateService {

    // Adjust to your real paths / names:
    private val extractionFolder = File("${context.filesDir.absolutePath}/tmp")
    private val basTempName = "firmware_raw.bas"
    private val manifestMarker = "manifest"        // file name contains this
    private val ospkgTarGzFileName = "firmware_imx6_dynamo.tar.gz" // example
    private val dOsTarGzFileName   = "ospkg.tar.gz"               // example
    private val dynamoAppTar       = "SOMApp.tar.gz"              // example

    // ---- read KEY/IV from the BAS (header text) if present ----
    private data class EncParams(val key: ByteArray, val iv: ByteArray)

    private fun readEncParamsFromBasHeader(bas: File): EncParams? {
        // We scan only the first few lines for:  KEY=... (hex)   and   IV=... (hex)
        return try {
            bas.inputStream().buffered().use { ins ->
                ins.reader(Charset.forName("UTF-8")).buffered().use { br ->
                    var keyHex: String? = null
                    var ivHex: String? = null
                    repeat(64) {
                        val line = br.readLine() ?: return@repeat
                        if (line.startsWith("KEY=", true)) keyHex = line.substringAfter("KEY=").trim()
                        if (line.startsWith("IV=",  true)) ivHex  = line.substringAfter("IV=").trim()
                        if (keyHex != null && ivHex != null) return EncParams(
                            AESEncryption.keyFromHex(keyHex!!),
                            AESEncryption.keyFromHex(ivHex!!)
                        )
                    }
                    null
                }
            }
        } catch (_: Throwable) {
            null
        }
    }

    // ---- 1) Extract BAS, decrypt any *.enc inside, then find and parse manifest ----
    override suspend fun extractFirmwareFile(
        firmwareFileInputStream: InputStream
    ): Outcome<SoftwareReleaseManifest> = withContext(Dispatchers.IO) {

        val basFile = File(context.filesDir, basTempName)
        // Write incoming stream to temp .bas
        basFile.outputStream().use { out ->
            firmwareFileInputStream.copyTo(out)
            out.flush()
        }

        // Clean extraction folder
        extractionFolder.listFiles()?.forEach { it.deleteRecursively() }
        extractionFolder.mkdirs()

        // TODO: replace with your real extractor
        val extractedOk: Boolean =
            try {
                // centralla.extractBasFile(basFile.absolutePath, extractionFolder.absolutePath)
                // or: dynamo.extractBasFile(...)
                dynamoExtractBasFile(basFile.absolutePath, extractionFolder.absolutePath)
            } catch (_: Throwable) { false }

        if (!extractedOk) {
            ProLog.w(MODULE_NAME, "Unable to extract bas file: ${basFile.name}.")
            return@withContext Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)
        }

        // Decrypt any *.enc files in-place (same name minus ".enc")
        val params = readEncParamsFromBasHeader(basFile)
        for (f in extractionFolder.listFiles().orEmpty()) {
            if (f.name.endsWith(".enc", true)) {
                val out = File(f.parentFile, f.name.removeSuffix(".enc"))
                if (out.exists()) out.delete()
                val key = params?.key ?: AESEncryption.keyFromHex("00112233445566778899AABBCCDDEEFF")
                val iv  = params?.iv  ?: ByteArray(16) { 0 } // replace with your real IV policy
                val ok = AESEncryption.decryptFileBlocking(
                    input = f, output = out, keyBytes = key, iv = iv, mode = AESEncryption.Mode.CBC_PKCS5
                )
                if (!ok) {
                    ProLog.e(MODULE_NAME, "decrypt failed: ${f.name}")
                    return@withContext Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)
                }
            }
        }

        // Find manifest (now plain text after decrypt step)
        val manifestFile = extractionFolder.listFiles()
            ?.firstOrNull { it.name.contains(manifestMarker, ignoreCase = true) }
            ?: return@withContext Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)

        // Parse manifest
        val releaseManifest = try {
            manifestFile.inputStream().use { ManifestParser().parse(it) }
        } catch (e: Exception) {
            ProLog.w(MODULE_NAME, "Unable to parse manifest file.")
            return@withContext Outcome.Error(e)
        }

        Outcome.Ok(releaseManifest)
    }

    // ---- 2) Your old compatibility routine, untouched except for using the extracted folder ----
    override suspend fun evaluateFirmwareCompatibility(
        activeOSVersion: DynamoYoctoOS,
        releaseManifest: SoftwareReleaseManifest
    ): Outcome<DynamoFirmwareCompatibilityStatus> = withContext(Dispatchers.IO) {

        var containsOS = false
        var osPkgFound = false
        var pkgOsVersion = DynamoYoctoOS.INVALID

        extractionFolder.listFiles()?.forEach { f ->
            if (f.name == ospkgTarGzFileName) {
                osPkgFound = true
                when (val ver = Dynamo.extractYoctoVersion(context, f)) {
                    is Outcome.Ok -> {
                        lastEvaluatedFwUpdateOsVersion = SemVer.fromString(ver.value)
                        containsOS = shouldContainOS(lastEvaluatedFwUpdateOsVersion)
                        pkgOsVersion = DynamoYoctoOS.fromVersion(lastEvaluatedFwUpdateOsVersion)
                    }
                    is Outcome.Error -> {
                        ProLog.e(MODULE_NAME, "Could not find yocto version in ${f.name}. Result: ${ver.error}")
                        return@withContext Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)
                    }
                }
                return@forEach
            } else if (f.name == dOsTarGzFileName) {
                when (val ver = Dynamo.extractYoctoVersion(context, f)) {
                    is Outcome.Ok -> {
                        lastEvaluatedFwUpdateOsVersion = SemVer.fromString(ver.value)
                        containsOS = shouldContainOS(lastEvaluatedFwUpdateOsVersion)
                        pkgOsVersion = DynamoYoctoOS.fromVersion(lastEvaluatedFwUpdateOsVersion)
                    }
                    is Outcome.Error -> {
                        ProLog.e(MODULE_NAME, "Could not find yocto version in ${f.name}. Result: ${ver.error}")
                        return@withContext Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)
                    }
                }
            }
        }

        if (pkgOsVersion == DynamoYoctoOS.INVALID) {
            extractionFolder.listFiles()?.forEach { f ->
                if (f.name == dynamoAppTar) {
                    pkgOsVersion = DynamoYoctoOS.DUNFELL
                    return@forEach
                }
            }
        }

        Outcome.Ok(
            if (!osPkgFound) DynamoFirmwareCompatibilityStatus.OSUpdateNotIncluded
            else if (!containsOS) DynamoFirmwareCompatibilityStatus.UpdateIncompatible
            else DynamoFirmwareCompatibilityStatus.Ok
        )
    }

    // ---- stubs & helpers you already had elsewhere ----

    // Replace this stub with your real extractor call.
    private fun dynamoExtractBasFile(basPath: String, outDir: String): Boolean {
        // e.g., call into your native or Java unzip/tar logic and return true/false
        // This stub always returns true to keep compilation.
        return true
    }

    private fun shouldContainOS(fwVersion: SemVer): Boolean =
        fwVersion >= SemVer.fromString(DynamoYoctoOS.DUNFELL.version)

    // You already have these in your project; left as type stubs so file compiles in isolation
    data class SoftwareReleaseManifest(val dummy: String = "")
    object ManifestParser { fun parse(input: InputStream): SoftwareReleaseManifest = SoftwareReleaseManifest() }
    data class SemVer(val major: Int, val minor: Int, val patch: Int) : Comparable<SemVer> {
        override fun compareTo(other: SemVer): Int =
            compareValuesBy(this, other, SemVer::major, SemVer::minor, SemVer::patch)
        companion object { fun fromString(s: String) = SemVer(0,0,0) }
    }
    enum class DynamoYoctoOS(val version: String) {
        INVALID("0.0.0"), DUNFELL("1.0.0");
        companion object { fun fromVersion(v: SemVer) = if (v >= SemVer(1,0,0)) DUNFELL else INVALID }
    }
    interface IDynamoFirmwareUpdateService {
        suspend fun extractFirmwareFile(firmwareFileInputStream: InputStream): Outcome<SoftwareReleaseManifest>
        suspend fun evaluateFirmwareCompatibility(
            activeOSVersion: DynamoYoctoOS,
            releaseManifest: SoftwareReleaseManifest
        ): Outcome<DynamoFirmwareCompatibilityStatus>
    }
    object Dynamo {
        fun extractYoctoVersion(context: Context, file: File): Outcome<String> = Outcome.Ok("1.0.0")
    }
    sealed class DynamoFirmwareCompatibilityStatus {
        data object Ok : DynamoFirmwareCompatibilityStatus()
        data object Evaluating : DynamoFirmwareCompatibilityStatus()
        data object Error : DynamoFirmwareCompatibilityStatus()
        data object UpdateIncompatible : DynamoFirmwareCompatibilityStatus()
        data object OSUpdateNotIncluded : DynamoFirmwareCompatibilityStatus()
    }
}
