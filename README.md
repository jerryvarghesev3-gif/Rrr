ll



// Returns (fileToRead, isTemp). If we decrypt, we create a temp with the target extension.
private fun resolvePlainIfEnc(
    context: Context,
    f: File
): Pair<File, Boolean> {
    val name = f.name.lowercase()

    // Not encrypted → just use as-is
    if (!name.endsWith(".enc")) return f to false

    // Decide the desired output extension based on the encrypted one
    val targetExt = when {
        name.endsWith(".tar.gz.enc") -> ".tar.gz"
        name.endsWith(".bin.enc")    -> ".bin"
        else                         -> "" // unknown — keep same stem
    }

    // Create a temp file next to app's cache/ files dir
    val out = if (targetExt.isNotEmpty()) {
        File.createTempFile(f.nameWithoutExtension.removeSuffix(".tar.gz"), targetExt,
            context.cacheDir)
    } else {
        File.createTempFile(f.nameWithoutExtension, null, context.cacheDir)
    }

    // --- Decrypt using your AES helper. If you removed AESEncryption, see stub below. ---
    val ok = AESEncryption.decryptFileBlocking(
        input = f,
        output = out
    )

    if (!ok) {
        // Cleanup and fall back to original error path
        runCatching { out.delete() }
        throw IllegalArgumentException("Decryption failed for ${f.name}")
    }

    return out to true
}
