kk



override suspend fun extractFirmwareFile(
    firmwareFileInputStream: InputStream
): Outcome<SoftwareReleaseManifest> {

    val basFile = File("${context.filesDir.absolutePath}/firmware.bas")
    // Write and CLOSE the BAS file before extraction
    basFile.outputStream().use { os ->
        firmwareFileInputStream.copyTo(os)
        os.fd.sync()                 // ensure bytes are on disk
    }

    // Clean extraction folder
    if (extractionFolder.exists()) {
        extractionFolder.listFiles()?.forEach { it.deleteRecursively() }
        extractionFolder.delete()
    }
    extractionFolder.mkdirs()

    // Now it’s safe to extract
    if (!dynamo.extractBasFile(basFile.absolutePath, extractionFolder.absolutePath)) {
        ProLog.w(MODULE_NAME, "Unable to extract bas file: ${basFile.name}")
        return Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)
    }

    // Find manifest (case-insensitive)
    val manifestFile = extractionFolder.listFiles()
        ?.firstOrNull { it.name.contains("manifest", ignoreCase = true) }
        ?: return Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)

    val releaseManifest = try {
        ManifestParser().parse(manifestFile.inputStream())
    } catch (e: Exception) {
        ProLog.w(MODULE_NAME, "Unable to parse manifest file.")
        return Outcome.Error(e)
    }

    return Outcome.Ok(releaseManifest)
}



