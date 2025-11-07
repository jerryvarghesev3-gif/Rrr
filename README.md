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











// DynamoFirmwareService.kt (or your *Service* file)
override suspend fun extractFirmwareFile(
    firmwareFileInputStream: InputStream
): Outcome<SoftwareReleaseManifest> {

    val basFile = File("${context.filesDir.absolutePath}/firmware_raw.bas")

    // Write the incoming stream to disk; we *only* manage the FileOutputStream here.
    try {
        FileOutputStream(basFile).use { os ->
            firmwareFileInputStream.copyTo(os)  // <-- consumes the stream once
            os.flush()
        }

        // Clean + prepare extraction folder
        extractionFolder.listFiles()?.forEach { it.deleteRecursively() }
        extractionFolder.mkdirs()

        // Run your existing BAS extractor
        if (!dynamo.extractBasFile(basFile.absolutePath, extractionFolder.absolutePath)) {
            ProLog.w(MODULE_NAME, "Unable to extract bas file: ${basFile.name}")
            return Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)
        }

        // Find and parse manifest
        val manifestFile = extractionFolder
            .listFiles()
            ?.firstOrNull { it.name.contains("manifest", ignoreCase = true) }
            ?: return Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)

        val releaseManifest = try {
            manifestFile.inputStream().use { ManifestParser().parse(it) }
        } catch (e: Exception) {
            ProLog.w(MODULE_NAME, "Unable to parse manifest file.")
            return Outcome.Error(e)
        }

        return Outcome.Ok(releaseManifest)
    } catch (t: Throwable) {
        ProLog.w(MODULE_NAME, "extractFirmwareFile failed: ${t.message}")
        return Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)
    }
}


// DynamoFirmwareBloc.kt (evaluateFirmwareCompatibility)
override suspend fun evaluateFirmwareCompatibility(
    activeOSVersion: DynamoYoctoOS,
    releaseManifest: SoftwareReleaseManifest?, // keep if you already store it; else compute below
): Outcome<DynamoFirmwareCompatibilityStatus> {

    update { s -> s.copy(firmwareCompatibilityStatus = DynamoFirmwareCompatibilityStatus.Evaluating) }

    return withContext(Dispatchers.IO) {
        // 1) Open a *fresh* InputStream and hand it to extractFirmwareFile
        val manifestOutcome: Outcome<SoftwareReleaseManifest> =
            contentResolver.openInputStream(basUri)?.use { basIs ->
                // basIs is consumed here and closed by .use afterwards
                eFirmwareService.extractFirmwareFile(basIs)
            } ?: Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)

        when (manifestOutcome) {
            is Outcome.Error -> {
                // early error path
                Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)
            }
            is Outcome.Ok -> {
                val manifest = manifestOutcome.value
                // cache for UI/debug if you keep it in state
                update { s -> s.copy(releaseManifest = manifest) }

                // 2) Continue with your existing compatibility logic.
                // IMPORTANT: Do NOT try to read the original InputStream again.
                eFirmwareService.evaluateFirmwareCompatibility(
                    bedStatusBloc.getSomOS(),
                    manifest
                )
            }
        }
    }
}
