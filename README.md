ll




// Read KEY/IV once from the BAS (or wherever you store them)
val params = readEncParamsFromBasHeader(basFile)
    ?: run {
        ProLog.e(MODULE_NAME, msg = "BAS header missing KEY/IV for ${f.name}")
        return Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)
    }

// Now call the decryptor
val ok = AESEncryption.decryptFileBlocking(
    input   = f,
    output  = out,
    keyBytes= params.key,
    iv      = params.iv,           // 16 bytes for CBC or GCM nonce length for GCM
    mode    = AESEncryption.Mode.CBC_PKCS5   // or .GCM if that’s what you use
)

if (!ok) {
    ProLog.e(MODULE_NAME, msg = "decrypt failed: ${f.name}")
    return Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)
}






object AESEncryption {

    enum class Mode { CBC_PKCS5, GCM }

    fun decryptFileBlocking(
        input: java.io.File,
        output: java.io.File,
        keyBytes: ByteArray,
        iv: ByteArray,
        mode: Mode = Mode.CBC_PKCS5
    ): Boolean {
        return try {
            // Build the secret key
            val secret = javax.crypto.spec.SecretKeySpec(keyBytes, "AES")

            // Choose transformation + parameter spec by mode
            val (transformation, params) = when (mode) {
                Mode.CBC_PKCS5 -> {
                    // IV must be 16 bytes
                    require(iv.size == 16) { "CBC requires 16-byte IV" }
                    "AES/CBC/PKCS5Padding" to javax.crypto.spec.IvParameterSpec(iv)
                }
                Mode.GCM -> {
                    // iv is the GCM nonce; tag length 16 bytes (128 bits) is common
                    val tagBits = 16 * 8
                    "AES/GCM/NoPadding" to javax.crypto.spec.GCMParameterSpec(tagBits, iv)
                }
            }

            // Init cipher for DECRYPT
            val cipher = javax.crypto.Cipher.getInstance(transformation)
            cipher.init(javax.crypto.Cipher.DECRYPT_MODE, secret, params)

            // Stream decrypt: input -> CipherInputStream -> output
            input.inputStream().use { fis ->
                javax.crypto.CipherInputStream(fis, cipher).use { cis ->
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
            true
        } catch (e: Throwable) {
            // Use whatever logger you have; falling back to println here
            try { ProLog.e(MODULE_NAME, msg = "decryptFileBlocking failed: ${e.message}") } catch (_: Throwable) {}
            false
        }
    }
}




