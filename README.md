hh


fun decryptFileToTemp(context: Context, encFile: File, plainSuffix: String): File {
    val tmpDir = File(context.filesDir, "tmp").apply { mkdirs() }
    val out = File(tmpDir, encFile.name.removeSuffix(".enc") + plainSuffix)

    FileInputStream(encFile).use { fin ->
        val iv = ByteArray(GCM_NONCE_LENGTH)
        require(fin.read(iv) == iv.size) { "IV missing in ${encFile.name}" }

        val cipher = Cipher.getInstance(ALGO)
        val spec = GCMParameterSpec(GCM_TAG_LENGTH * 8, iv)
        cipher.init(Cipher.DECRYPT_MODE, encryptionKey, spec)

        CipherInputStream(fin, cipher).use { cin ->
            FileOutputStream(out).use { fout ->
                val buf = ByteArray(64 * 1024)
                var n: Int
                while (cin.read(buf).also { n = it } >= 0) {
                    if (n > 0) fout.write(buf, 0, n)
                }
                fout.fd.sync() // <- this must be INSIDE FileOutputStream.use
            }
        }
    }

    require(out.length() > 0L) { "Decrypted empty: ${out.path}" }
    return out
}
