ll



// Encryption.kt (keep object name if you already have it)
object AESEncryption {
    // transformation must match packer (do NOT change unless encryption side changes)
    private const val TRANSFORM = "AES/CBC/PKCS5Padding"

    /** Decrypts ENC -> TAR.GZ. Returns true on success. */
    fun decryptFileBlocking(
        input: File,
        output: File,
        keyBytes: ByteArray,
        iv: ByteArray
    ): Boolean {
        require(iv.size == 16) { "IV must be 16 bytes" }
        val secret = javax.crypto.spec.SecretKeySpec(keyBytes, "AES")
        val cipher  = javax.crypto.Cipher.getInstance(TRANSFORM)
        cipher.init(javax.crypto.Cipher.DECRYPT_MODE, secret, javax.crypto.spec.IvParameterSpec(iv))

        input.inputStream().use { fis ->
            output.outputStream().use { fos ->
                javax.crypto.CipherOutputStream(fos, cipher).use { cos ->
                    val buf = ByteArray(64 * 1024)
                    while (true) {
                        val n = fis.read(buf)
                        if (n <= 0) break
                        cos.write(buf, 0, n)
                    }
                }
            }
        }
        return true
    }
}






override suspend fun extractFirmwareFile(
    firmwareFileInputStream: InputStream
): Outcome<SoftwareReleaseManifest> {

    val basFile = File("${context.filesDir.absolutePath}/firmware_raw.bas")
    val os = basFile.outputStream()
    return try {
        // write BAS once
        firmwareFileInputStream.copyTo(os)
        os.flush()

        // Clean extraction folder
        extractionFolder.listFiles()?.forEach { it.deleteRecursively() }
        extractionFolder.mkdirs()

        // If BAS contains encrypted payloads, decrypt them to plain *.tar.gz files first
        // (so your existing code that looks for *.tar.gz keeps working as-is).
        // This assumes your 'extractBasFile' simply dumps files to 'extractionFolder'.
        if (!centrella.extractBasFile(basFile.absolutePath, extractionFolder.absolutePath)) {
            ProLog.w(MODULE_NAME, msg = "Unable to extract bas file: ${basFile.name}.")
            return Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)
        }

        // decrypt any *.enc members in-place (-> same name without .enc)
        extractionFolder.listFiles()?.forEach { f ->
            if (f.name.endsWith(".enc", ignoreCase = true)) {
                val out = File(f.parentFile, f.name.removeSuffix(".enc")) // e.g. *.tar.gz
                val params = readEncParamsFromBasHeader(basFile)
                    ?: return Outcome.Error(DynamoFirmwareCompatibilityStatus.Error).also {
                        ProLog.e(MODULE_NAME, "BAS header missing KEY/IV for ${f.name}")
                    }
                val ok = AESEncryption.decryptFileBlocking(
                    input   = f,
                    output  = out,
                    keyBytes= params.key,
                    iv      = params.iv
                )
                if (!ok) {
                    ProLog.e(MODULE_NAME, "decrypt failed: ${f.name}")
                    return Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)
                }
            }
        }

        // Now find manifest (already plain text after decrypt)
        val manifestFile = extractionFolder
            .listFiles()
            ?.firstOrNull { it.name.contains("manifest", ignoreCase = true) }
            ?: return Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)

        val releaseManifest = try {
            manifestFile.inputStream().use { ManifestParser().parse(it) }
        } catch (e: Exception) {
            ProLog.w(MODULE_NAME, msg = "Unable to parse manifest file.", e = e)
            return Outcome.Error(e)
        }

        Outcome.Ok(releaseManifest)

    } catch (e: Exception) {
        ProLog.e(MODULE_NAME, msg = "extractFirmwareFile failed", e = e)
        Outcome.Error(e)
    } finally {
        try { os.close() } catch (_: Throwable) {}
    }
}










private data class EncParams(val key: ByteArray, val iv: ByteArray)

private fun readEncParamsFromBasHeader(bas: File): EncParams? {
    // Example: first few KB contain lines like: KEY=..., IV=...
    bas.inputStream().buffered().use { ins ->
        val br = ins.reader(Charsets.UTF_8).buffered()
        repeat(64) { // scan limited lines
            val line = br.readLine() ?: return@repeat
            if (line.startsWith("KEY=")) {
                val keyHex = line.substringAfter("KEY=").trim()
                val ivHex  = br.readLine()?.substringAfter("IV=")?.trim() ?: return null
                return EncParams(keyHex.hexToBytes(), ivHex.hexToBytes())
            }
            if (line.startsWith("----")) return null // header done
        }
    }
    return null
}






