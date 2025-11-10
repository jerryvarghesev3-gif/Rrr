ll



override suspend fun extractFirmwareFile(
    firmwareFileInputStream: InputStream
): Outcome<SoftwareReleaseManifest> = withContext(Dispatchers.IO) {
    // 1) Write incoming BAS to a temp file and CLOSE it right away
    val basFile = File("${context.filesDir.absolutePath}/firmware.bas")
    try {
        basFile.outputStream().use { out ->
            firmwareFileInputStream.copyTo(out)
            out.flush() // (redundant with use but explicit)
        }
    } catch (e: Throwable) {
        ProLog.e(MODULE_NAME, "Error writing bas file: ${e.message}")
        return@withContext Outcome.Error(e)
    }

    // 2) Clean extraction folder
    extractionFolder.listFiles()?.forEach { it.deleteRecursively() }
    extractionFolder.mkdirs()

    // 3) Extract BAS (no decryption of contents; we keep *.tar.gz.enc as-is)
    val extractedOk = try {
        // your native/legacy extractor
        dynamo.extractBasFile(basFile.absolutePath, extractionFolder.absolutePath)
    } catch (_: Throwable) { false }

    if (!extractedOk) {
        ProLog.w(MODULE_NAME, "Unable to extract bas file: ${basFile.name}")
        return@withContext Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)
    }

    // 4) Find the manifest (now plain text after unzip; we do NOT touch *.tar.gz.enc)
    val manifestFile = extractionFolder.listFiles()
        ?.firstOrNull { it.name.contains("manifest", ignoreCase = true) }
        ?: return@withContext Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)

    // 5) Parse manifest
    val releaseManifest = try {
        manifestFile.inputStream().use { ManifestParser().parse(it) }
    } catch (e: Exception) {
        ProLog.w(MODULE_NAME, "Unable to parse manifest file.")
        return@withContext Outcome.Error(e)
    }

    Outcome.Ok(releaseManifest)
}
