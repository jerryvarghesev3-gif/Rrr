ll




fun decryptFileBlocking(
    input: File,
    output: File,
    keyBytes: ByteArray,      // ✅ REQUIRED
    iv: ByteArray,
    mode: Mode = Mode.CBC_PKCS5
): Boolean {
    return try {
        val secret = SecretKeySpec(keyBytes, "AES")

        val cipher = when (mode) {
            Mode.CBC_PKCS5 -> Cipher.getInstance("AES/CBC/PKCS5Padding").apply {
                require(iv.size == 16)
                init(Cipher.DECRYPT_MODE, secret, IvParameterSpec(iv))
            }
            Mode.GCM -> Cipher.getInstance("AES/GCM/NoPadding").apply {
                init(Cipher.DECRYPT_MODE, secret, GCMParameterSpec(16 * 8, iv))
            }
        }

        FileInputStream(input).use { fis ->
            CipherInputStream(fis, cipher).use { cis ->
                FileOutputStream(output).use { fos ->
                    cis.copyTo(fos)
                }
            }
        }
        true
    } catch (t: Throwable) {
        ProLog.e(MODULE_NAME, "decryptFileBlocking failed: ${t.message}")
        false
    }
}

