ll



override suspend fun extractFirmwareFile(
    firmwareFileInputStream: InputStream
): Outcome<SoftwareReleaseManifest> {

    // 0) Persist incoming .bas to app storage
    val basFile = File("${context.filesDir.absolutePath}/firmware_raw.bas")
    val os = basFile.outputStream()
    try {
        firmwareFileInputStream.copyTo(os)
    } finally {
        os.close()
    }

    // 1) Clean extraction folder
    if (extractionFolder.listFiles()?.isNotEmpty() == true) {
        extractionFolder.listFiles()?.forEach { it.deleteRecursively() }
    }
    extractionFolder.mkdirs()

    // 2) Extract the .bas (DO NOT decrypt the .bas)
    if (!dynamo.extractBasFile(basFile.absolutePath, extractionFolder.absolutePath)) {
        ProLog.w(MODULE_NAME, "Unable to extract bas file: ${basFile.name}")
        return Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)
    }

    // 3) AFTER extraction, decrypt only *.enc payloads
    //    Example: SOMApp.tar.gz.enc  -> SOMApp.tar.gz
    //             firmware_imx6.tar.gz.enc -> firmware_imx6.tar.gz
    extractionFolder.listFiles()?.forEach { f ->
        if (f.isFile && f.name.endsWith(".enc", ignoreCase = true)) {
            val plainName = f.name.removeSuffix(".enc") // keeps .tar.gz, .bin, etc.
            val outFile = File(f.parentFile, plainName)
            try {
                // your existing decrypt function that expects IV at start of the .enc file
                decryptFileToTemp(context, f, plainSuffix = "")  // or decryptFile(f, outFile)
                // If your decrypt API returns the output file, use that instead:
                // val produced = decryptFileToTemp(context, f, "")
                // produced.renameTo(outFile)
                // Replace source with decrypted
                if (outFile.exists()) f.delete()
            } catch (e: Exception) {
                ProLog.w(MODULE_NAME, "Decryption failed for ${f.name}: ${e.message}")
                return Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)
            }
        }
    }

    // 4) Locate and parse manifest (same as before)
    val manifestFile = extractionFolder.listFiles()?.firstOrNull { it.name.contains("manifest", true) }
        ?: return Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)

    val releaseManifest = try {
        ManifestParser().parse(manifestFile.inputStream())
    } catch (e: Exception) {
        ProLog.w(MODULE_NAME, "Unable to parse manifest file: ${e.message}")
        return Outcome.Error(e)
    }

    return Outcome.Ok(releaseManifest)
}
