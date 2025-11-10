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











// Decrypts *.tar.gz.enc / *.bin.enc to a temp file (next to app cache) and
// returns Pair(<file to read next>, <wasDecrypted>).
// If the input isn't *.enc, it just returns the original file and false.
private fun resolvePlainIfEnc(
    context: Context,
    f: File,
    basFile: File
): Pair<File, Boolean> {

    val name = f.name.lowercase()

    // decide the desired output extension after decrypt
    val targetExt = when {
        name.endsWith(".tar.gz.enc") -> ".tar.gz"
        name.endsWith(".bin.enc")    -> ".bin"
        else                         -> ""              // unknown -> keep same stem
    }

    // if it's not encrypted, nothing to do
    if (!name.endsWith(".enc")) {
        return f to false
    }

    // create a temp output next to app's cache/files dir
    val out = if (targetExt.isNotEmpty()) {
        File.createTempFile(f.nameWithoutExtension, targetExt, context.cacheDir)
    } else {
        // keep same stem if we don't know the final suffix
        File.createTempFile(f.nameWithoutExtension, null, context.cacheDir)
    }.apply { if (exists()) delete() }

    // read KEY/IV from BAS header (adapt if you store them elsewhere)
    val params = AESEncryption.readEncParamsFromBasHeader(basFile) ?: run {
        ProLog.e(MODULE_NAME, "BAS header missing KEY/IV for ${f.name}")
        return f to false
    }

    // decrypt using your AES helper
    val ok = AESEncryption.decryptFileBlocking(
        input   = f,
        output  = out,
        keyBytes= params.key,
        iv      = params.iv,
        mode    = AESEncryption.Mode.CBC_PKCS5   // or GCM if your producer uses it
    )

    return if (ok) out to true else f to false
}

