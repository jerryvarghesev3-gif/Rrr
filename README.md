hh
// AESEncryption.kt
object AESEncryption {
    // Make these visible to the helper (or keep private and inline the literals)
    const val ALGO = "AES/GCM/NoPadding"
    const val GCM_TAG_LENGTH = 16
    const val GCM_NONCE_LENGTH = 12

    // encryptionKey = your existing SecretKeySpec(...)
    val encryptionKey = /* your existing key */

    /**
     * Decrypts [encFile] whose first bytes are the IV, then ciphertext+tag,
     * and writes a temp plain file with [plainSuffix] under /files/tmp.
     */
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
                    while (cin.read(buf).also { n = it } >= 0) if (n > 0) fout.write(buf, 0, n)
                    fout.fd.sync()
                }
            }
        }
        require(out.length() > 0L) { "Decrypted empty: ${out.path}" }
        return out
    }
}



// Tar/util file (where DextractYoctoVersion lives)

private fun resolvePlainIfEnc(context: Context, f: File): Pair<File, Boolean> {
    val name = f.name.lowercase()
    return when {
        name.endsWith(".tar.gz.enc") ->
            AESEncryption.decryptFileToTemp(context, f, ".tar.gz") to true
        name.endsWith(".bin.enc") ->
            AESEncryption.decryptFileToTemp(context, f, ".bin") to true
        else -> f to false
    }
}



fun DextractYoctoVersion(context: Context, targzFile: File): Outcome<String> {
    var temp: File? = null
    try {
        val (toRead, isTemp) = resolvePlainIfEnc(context, targzFile)
        if (isTemp) temp = toRead

        // --- your existing code below, but use 'toRead' instead of 'targzFile' when opening ---
        // when (val hVersion = searchTarGzFile(toRead, "h-version")) { ... }
        // lines.forEach { ... regex ... } etc.

        return outcome // exactly as you had

    } catch (t: Throwable) {
        return Outcome.Error(t.message ?: "Yocto version parse failed")
    } finally {
        // Clean the local temp only (never delete originals)
        temp?.delete()
    }
}



val ver = DextractYoctoVersion(context, it /* the file from folder */)
