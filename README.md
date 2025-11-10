ll




import java.io.*
import java.nio.charset.Charset
import javax.crypto.Cipher
import javax.crypto.CipherInputStream
import javax.crypto.spec.GCMParameterSpec
import javax.crypto.spec.IvParameterSpec
import javax.crypto.spec.SecretKeySpec

// pick anything short; it's only for logging
private const val MODULE_NAME = "FIRMWARE"

object AESEncryption {
    enum class Mode { CBC_PKCS5, GCM }

    private const val TRANSFORM_CBC = "AES/CBC/PKCS5Padding"
    private const val TRANSFORM_GCM = "AES/GCM/NoPadding"
    private const val GCM_TAG_LEN_BYTES = 16

    data class EncParams(val key: ByteArray, val iv: ByteArray)

    fun keyFromHex(hex: String): ByteArray =
        hex.trim().replace("\\s".toRegex(), "")
            .chunked(2).map { it.toInt(16).toByte() }.toByteArray()

    /** Scan first ~64 lines of the BAS for
     *  lines like "KEY=001122..." and "IV=A1B2C3..." */
    fun readEncParamsFromBasHeader(basFile: File): EncParams? {
        return try {
            basFile.inputStream().buffered().use { ins ->
                ins.reader(Charset.forName("UTF-8")).buffered().use { br ->
                    var keyHex: String? = null
                    var ivHex: String? = null
                    repeat(64) {
                        val line = br.readLine() ?: return@repeat
                        if (line.startsWith("KEY=", ignoreCase = true))
                            keyHex = line.substringAfter("KEY=").trim()
                        if (line.startsWith("IV=", ignoreCase = true))
                            ivHex = line.substringAfter("IV=").trim()
                        if (keyHex != null && ivHex != null) {
                            return EncParams(keyFromHex(keyHex!!), keyFromHex(ivHex!!))
                        }
                    }
                }
            }
            null
        } catch (_: Throwable) { null }
    }

    /** Blocking decrypt helper used by your extractor. */
    fun decryptFileBlocking(
        input: File,
        output: File,
        keyBytes: ByteArray,
        iv: ByteArray,
        mode: Mode = Mode.CBC_PKCS5
    ): Boolean {
        return try {
            val secret = SecretKeySpec(keyBytes, "AES")
            val cipher = when (mode) {
                Mode.CBC_PKCS5 -> Cipher.getInstance(TRANSFORM_CBC).apply {
                    init(Cipher.DECRYPT_MODE, secret, IvParameterSpec(iv))
                }
                Mode.GCM -> Cipher.getInstance(TRANSFORM_GCM).apply {
                    init(Cipher.DECRYPT_MODE, secret, GCMParameterSpec(GCM_TAG_LEN_BYTES * 8, iv))
                }
            }
            FileInputStream(input).use { fis ->
                CipherInputStream(fis, cipher).use { cis ->
                    FileOutputStream(output).use { fos ->
                        cis.copyTo(fos)
                        fos.flush()
                    }
                }
            }
            true
        } catch (e: Throwable) {
            ProLog.e(MODULE_NAME, msg = "decryptFileBlocking failed: ${e.message}")
            false
        }
    }
}






/** If f is *.enc, decrypt to a temp file and return (plainFile, true),
 *  else return (f, false).  basFile is the header we read KEY/IV from. */
private fun resolvePlainIfEnc(
    context: Context,
    f: File,
    basFile: File
): Pair<File, Boolean> {
    val name = f.name.lowercase()

    val targetExt = when {
        name.endsWith(".tar.gz.enc") -> ".tar.gz"
        name.endsWith(".bin.enc")    -> ".bin"
        else                         -> ""            // keep same stem
    }

    // Create a temp output next to app cache
    val out = if (targetExt.isNotEmpty())
        File.createTempFile(f.nameWithoutExtension, targetExt, context.cacheDir)
    else
        File.createTempFile(f.nameWithoutExtension, null, context.cacheDir)

    // If not encrypted, just use source as-is
    if (!name.endsWith(".enc")) return f to false

    // Read KEY/IV from BAS header
    val params = AESEncryption.readEncParamsFromBasHeader(basFile) ?: run {
        ProLog.e(MODULE_NAME, msg = "BAS header missing KEY/IV for ${f.name}")
        return f to false
    }

    val ok = AESEncryption.decryptFileBlocking(
        input   = f,
        output  = out,
        keyBytes= params.key,
        iv      = params.iv,
        mode    = AESEncryption.Mode.CBC_PKCS5 // or GCM if your producer uses it
    )
    return if (ok) out to true else f to false
}











// Put this INSIDE object AESEncryption
fun decryptFileBlocking(
    input: File,
    output: File,
    iv: ByteArray,
    mode: Mode = Mode.GCM   // default to your current scheme
): Boolean {
    return try {
        val cipher = when (mode) {
            Mode.CBC_PKCS5 -> {
                require(iv.size == 16) { "CBC IV must be 16 bytes" }
                javax.crypto.Cipher.getInstance(TRANSFORM_CBC).apply {
                    init(javax.crypto.Cipher.DECRYPT_MODE, encryptionKey,
                        javax.crypto.spec.IvParameterSpec(iv))
                }
            }
            Mode.GCM -> {
                require(iv.size == GCM_NONCE_LENGTH) { "GCM IV (nonce) must be $GCM_NONCE_LENGTH bytes" }
                javax.crypto.Cipher.getInstance(algorithm).apply {
                    init(javax.crypto.Cipher.DECRYPT_MODE, encryptionKey,
                        javax.crypto.spec.GCMParameterSpec(GCM_TAG_LENGTH * 8, iv))
                }
            }
        }

        java.io.FileInputStream(input).use { fis ->
            javax.crypto.CipherInputStream(fis, cipher).use { cis ->
                java.io.FileOutputStream(output).use { fos ->
                    cis.copyTo(fos)
                    fos.flush()
                }
            }
        }
        true
    } catch (t: Throwable) {
        ProLog.e(MODULE_NAME, msg = "decryptFileBlocking failed: ${t.message}")
        false
    }
}



