ll


// AESEncryption.kt
object AESEncryption {
    enum class Mode { CBC_PKCS5, GCM }

    // ---- BLOCKING overload (no suspend) ----
    fun decryptFileBlocking(
        input: java.io.File,
        output: java.io.File,
        keyBytes: ByteArray = keyFromHex(DEFAULT_KEY_HEX),
        iv: ByteArray? = null,
        mode: Mode = Mode.CBC_PKCS5
    ): Boolean {
        var fis: java.io.FileInputStream? = null
        var fos: java.io.FileOutputStream? = null
        var cis: javax.crypto.CipherInputStream? = null
        return try {
            fis = java.io.FileInputStream(input)

            val realIv = iv ?: ByteArray(16).also {
                val n = fis.read(it)
                if (n != 16) throw IllegalArgumentException("IV header not present")
            }

            val key = javax.crypto.spec.SecretKeySpec(keyBytes, "AES")
            val cipher = when (mode) {
                Mode.CBC_PKCS5 -> javax.crypto.Cipher.getInstance("AES/CBC/PKCS5Padding")
                    .apply { init(javax.crypto.Cipher.DECRYPT_MODE, key,
                        javax.crypto.spec.IvParameterSpec(realIv)) }
                Mode.GCM -> javax.crypto.Cipher.getInstance("AES/GCM/NoPadding")
                    .apply { init(javax.crypto.Cipher.DECRYPT_MODE, key,
                        javax.crypto.spec.GCMParameterSpec(128, realIv)) }
            }

            fos = java.io.FileOutputStream(output)
            cis = javax.crypto.CipherInputStream(fis, cipher)

            val buf = ByteArray(64 * 1024)
            while (true) {
                val n = cis.read(buf)
                if (n <= 0) break
                fos.write(buf, 0, n)
            }
            fos.flush()
            true
        } catch (t: Throwable) {
            android.util.Log.e("AESEncryption", "decryptFileBlocking failed: ${t.message}", t)
            false
        } finally {
            try { cis?.close() } catch (_: Throwable) {}
            try { fos?.close() } catch (_: Throwable) {}
            try { fis?.close() } catch (_: Throwable) {}
        }
    }

    // keep the suspend version if you need it elsewhere
    suspend fun decryptFile(
        input: java.io.File,
        output: java.io.File,
        keyBytes: ByteArray = keyFromHex(DEFAULT_KEY_HEX),
        iv: ByteArray? = null,
        mode: Mode = Mode.CBC_PKCS5
    ): Boolean = kotlinx.coroutines.withContext(kotlinx.coroutines.Dispatchers.IO) {
        decryptFileBlocking(input, output, keyBytes, iv, mode)
    }

    // util
    private const val DEFAULT_KEY_HEX = "00112233445566778899AABBCCDDEEFF" // <-- replace
    fun keyFromHex(hex: String): ByteArray {
        val c = hex.trim().replace(" ", "")
        val out = ByteArray(c.length / 2)
        var i = 0
        while (i < c.length) {
            out[i / 2] = (((c[i].digitToInt(16) shl 4) or c[i + 1].digitToInt(16))).toByte()
            i += 2
        }
        return out
    }
}








// your minimal helper
fun File.isEnc() = name.endsWith(".enc", ignoreCase = true)

fun File.decryptedIfNeeded(): File {
    if (!isEnc()) return this
    val out = File(parentFile, name.removeSuffix(".enc"))
    if (out.exists()) out.delete()

    val ok = AESEncryption.decryptFileBlocking(
        input = this,
        output = out,
        // keyBytes = AESEncryption.keyFromHex(BuildConfig.FW_KEY_HEX), // if you store it
        iv = null,                                   // null → read 16-byte IV header
        mode = AESEncryption.Mode.CBC_PKCS5          // or GCM, whichever you use
    )
    if (!ok) throw IllegalArgumentException("Decryption failed for ${this.name}")
    return out
}k
