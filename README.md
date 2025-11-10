kk



private const val MODULE_NAME = "DynamoFW"

enum class Mode { CBC_PKCS5, GCM }

// crypto transforms (strings are what Cipher.getInstance expects)
private const val TRANSFORM_CBC = "AES/CBC/PKCS5Padding"
private const val TRANSFORM_GCM = "AES/GCM/NoPadding"

// GCM tag length given in *bytes* (Cipher API wants bits -> we multiply by 8 later)
private const val GCM_TAG_LEN_BYTES = 16





object AESEncryption {

    data class EncParams(val key: ByteArray, val iv: ByteArray)

    /** Parse KEY=… and IV=… (hex) from the first few lines of the .bas file */
    fun readEncParamsFromBasHeader(bas: File): EncParams? {
        return try {
            bas.inputStream().buffered().use { ins ->
                ins.reader(Charset.forName("UTF-8")).buffered().use { br ->
                    var keyHex: String? = null
                    var ivHex: String? = null
                    repeat(64) {      // scan just a small header window
                        val line = br.readLine() ?: return@repeat
                        if (line.startsWith("KEY=", ignoreCase = true)) {
                            keyHex = line.substringAfter("KEY=").trim()
                        }
                        if (line.startsWith("IV=", ignoreCase = true)) {
                            ivHex = line.substringAfter("IV=").trim()
                        }
                        if (keyHex != null && ivHex != null) {
                            return EncParams(keyFromHex(keyHex!!), keyFromHex(ivHex!!))
                        }
                    }
                }
            }
            null
        } catch (_: Throwable) {
            null
        }
    }

    /** Hex → bytes (no spaces, even length) */
    fun keyFromHex(hex: String): ByteArray {
        val c = hex.trim().replace("\\s".toRegex(), "")
        require(c.length % 2 == 0) { "hex length must be even" }
        val out = ByteArray(c.length / 2)
        var i = 0
        var j = 0
        while (i < c.length) {
            out[j++] = (((c[i].digitToInt(16)) shl 4) or (c[i + 1].digitToInt(16))).toByte()
            i += 2
        }
        return out
    }

    /**
     * Decrypt a file to another file.
     * Pass the **correct** key/iv/mode that your producer used, otherwise you'll get WRONG_FINAL_BLOCK_LENGTH.
     */
    fun decryptFileBlocking(
        input: File,
        output: File,
        keyBytes: ByteArray,
        iv: ByteArray,
        mode: Mode = Mode.CBC_PKCS5
    ): Boolean {
        return try {
            val secret = javax.crypto.spec.SecretKeySpec(keyBytes, "AES")
            val cipher = when (mode) {
                Mode.CBC_PKCS5 -> javax.crypto.Cipher.getInstance(TRANSFORM_CBC).apply {
                    require(iv.size == 16) { "CBC IV must be 16 bytes" }
                    init(javax.crypto.Cipher.DECRYPT_MODE, secret, javax.crypto.spec.IvParameterSpec(iv))
                }
                Mode.GCM -> javax.crypto.Cipher.getInstance(TRANSFORM_GCM).apply {
                    init(
                        javax.crypto.Cipher.DECRYPT_MODE,
                        secret,
                        javax.crypto.spec.GCMParameterSpec(GCM_TAG_LEN_BYTES * 8, iv)
                    )
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
            ProLog.e(MODULE_NAME, "decryptFileBlocking failed: ${t.message}")
            false
        }
    }
}





/** If `f` ends with .enc, decrypt it next to cache dir and return (out, true); else (f, false) */
private fun resolvePlainIfEnc(
    context: Context,
    f: File,
    basFile: File,                // we must know where to read KEY/IV
    mode: Mode = Mode.CBC_PKCS5   // switch to Mode.GCM if your producer is GCM
): Pair<File, Boolean> {

    val name = f.name.lowercase()

    // not encrypted? return the original file as-is
    if (!name.endsWith(".enc")) return f to false

    // choose desired plaintext extension by encrypted type
    val targetExt = when {
        name.endsWith(".tar.gz.enc") -> ".tar.gz"
        name.endsWith(".bin.enc")    -> ".bin"
        else                         -> ""           // keep same stem if unknown
    }

    // temp output next to cache
    val out = if (targetExt.isNotEmpty()) {
        File.createTempFile(f.nameWithoutExtension.removeSuffix(".tar.gz"), targetExt, context.cacheDir)
    } else {
        File.createTempFile(f.nameWithoutExtension, null, context.cacheDir)
    }

    // read KEY/IV
    val params = AESEncryption.readEncParamsFromBasHeader(basFile)
        ?: run {
            ProLog.e(MODULE_NAME, "BAS header missing KEY/IV for ${f.name}")
            return f to false
        }

    // decrypt
    val ok = AESEncryption.decryptFileBlocking(
        input   = f,
        output  = out,
        keyBytes= params.key,
        iv      = params.iv,
        mode    = mode
    )
    return if (ok) out to true else f to false
}





fun dynamoExtractYoctoVersion(
    context: Context,
    targzFile: File,
    basFile: File,                      // <-- add this
    mode: Mode = Mode.CBC_PKCS5
): Outcome<String> {
    var temp: File? = null
    return try {
        val (toRead, wasTemp) = resolvePlainIfEnc(context, targzFile, basFile, mode)
        if (wasTemp) temp = toRead

        // your existing search logic …
        val hillromVersion = searchTarGzFile(toRead, "hillrom-version")
        when (hillromVersion) {
            is Outcome.Ok -> {
                val lines = hillromVersion.value.decodeToString().split('\n').map { it.trim() }
                for (line in lines) {
                    if (line.contains("Version:", ignoreCase = true)) {
                        val match = Regex("\\d+(\\.\\d+)+").find(line)?.value
                        return if (match != null) Outcome.Ok(match) else Outcome.Error("Failed to decode OS version.")
                    }
                }
                Outcome.Error("Failed to decode OS version.")
            }
            is Outcome.Error -> Outcome.Error(hillromVersion.error as String)
        }
    } catch (t: Throwable) {
        Outcome.Error("Yocto version parse failed: ${t.message}")
    } finally {
        // only delete temp
        temp?.delete()
    }
}





when (val ver = dynamoExtractYoctoVersion(context, f, basFile /* <- this one */)) {
    is Outcome.Ok   -> { /* your old success path */ }
    is Outcome.Error-> { /* your old error path */ }
}




private const val manifestMarker = "manifest"




val manifestFile = extractionFolder
    .listFiles()
    ?.firstOrNull { it.name.contains(manifestMarker, ignoreCase = true) }
    ?: return Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)

val releaseManifest = try {
    manifestFile.inputStream().use { ManifestParser().parse(it) }
} catch (e: Exception) {
    ProLog.w(MODULE_NAME, "Unable to parse manifest file.")
    return Outcome.Error(e)
}





val extractedOk = try {
    // use whichever one is your actual implementation
    centralla.extractBasFile(basFile.absolutePath, extractionFolder.absolutePath)
    // or: dynamo.extractBasFile(...)
} catch (_: Throwable) {
    false
}
if (!extractedOk) {
    ProLog.w(MODULE_NAME, "Unable to extract bas file: ${basFile.name}.")
    return withContext(Dispatchers.IO) { Outcome.Error(DynamoFirmwareCompatibilityStatus.Error) }
}







