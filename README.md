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
















fun dynamoExtractYoctoVersion(
    context: Context,
    targzFile: File,
    basFile: File       // <— add BAS here so resolvePlainIfEnc can pull KEY/IV
): Outcome<String> {
    var temp: File? = null

    return try {
        // 1) If *.tar.gz.enc or *.bin.enc, decrypt next to cache and read that.
        val (toRead, isTemp) = resolvePlainIfEnc(context, targzFile, basFile)
        if (isTemp) temp = toRead

        var outcome: Outcome<String> = Outcome.Error("${targzFile.name} invalid input")

        // 2) We only parse the OS package tarballs
        if (targzFile.name == dynamoosTarGzFileName || targzFile.name == dynamamosTarGzFileName) {
            // 3) Inside the tar.gz look for the "hillrom-version" file and read its text
            when (val hillromVersion = searchTarGzFile(toRead, searchFileName = "hillrom-version")) {
                is Outcome.Ok -> {
                    val lines = hillromVersion.value.decodeToString()
                        .split('\n')
                        .map { it.trim() }

                    // 4) Find a line that contains "Version:" and extract X.Y or X.Y.Z…
                    var found: String? = null
                    lines.forEach { line ->
                        if (line.contains("Version:", ignoreCase = true)) {
                            val regex = Regex("""\d+(?:\.\d+)+""")
                            found = regex.find(line)?.value
                            return@forEach
                        }
                    }

                    outcome = if (found != null) {
                        ProLog.i(MODULE_NAME, msg = "${targzFile.name} OS version = $found")
                        Outcome.Ok(found!!)
                    } else {
                        ProLog.e(MODULE_NAME, msg = "Failed to decode OS version.")
                        Outcome.Error("Failed to decode OS version.")
                    }
                }
                is Outcome.Error -> {
                    outcome = Outcome.Error(hillromVersion.error as String)
                }
            }
        }

        outcome
    } catch (t: Throwable) {
        Outcome.Error("Yocto version parse failed: ${t.message}")
    } finally {
        // Only delete our temp (never the original)
        temp?.delete()
    }
}

