ll



override suspend fun extractFirmwareFile(
    firmwareFileInputStream: InputStream
): Outcome<SoftwareReleaseManifest> {

    val basFile = File("${context.filesDir.absolutePath}/firmware_raw.bas")

    return try {
        // 1) Persist stream to disk, durably
        basFile.outputStream().use { out ->
            firmwareFileInputStream.copyTo(out)
            out.fd.sync()                // make sure bytes hit storage
        }
        if (basFile.length() <= 0L) {
            ProLog.w(MODULE_NAME, "Empty BAS after copy: ${basFile.absolutePath}")
            return Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)
        }

        // 2) Clean extraction folder
        extractionFolder.listFiles()?.forEach { it.deleteRecursively() }
        extractionFolder.mkdirs()

        // 3) Extract BAS (zip/tar handled inside your dynamo.extractBasFile)
        val ok = dynamo.extractBasFile(
            basFile.absolutePath,
            extractionFolder.absolutePath
        )
        if (!ok) {
            ProLog.w(MODULE_NAME, "Unable to extract bas file: ${basFile.name}")
            return Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)
        }

        // 4) Locate manifest (case-insensitive, first match)
        val manifestFile = extractionFolder
            .listFiles()
            ?.firstOrNull { it.name.contains("manifest", ignoreCase = true) }
            ?: return Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)

        // 5) Parse manifest with safe resource handling
        val releaseManifest = try {
            manifestFile.inputStream().use { ManifestParser().parse(it) }
        } catch (e: Exception) {
            ProLog.w(MODULE_NAME, "Unable to parse manifest file. ${e.message}")
            return Outcome.Error(e)
        }

        Outcome.Ok(releaseManifest)
    } finally {
        try { firmwareFileInputStream.close() } catch (_: Throwable) {}
    }
}

