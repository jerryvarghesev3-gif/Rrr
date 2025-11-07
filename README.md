ll





// In *Service* (unchanged from last fix)
override suspend fun extractFirmwareFile(
    firmwareFileInputStream: InputStream
): Outcome<SoftwareReleaseManifest> {

    val basFile = File("${context.filesDir.absolutePath}/firmware_raw.bas")

    return try {
        // write the incoming stream to disk; only close the FileOutputStream
        FileOutputStream(basFile).use { os ->
            firmwareFileInputStream.copyTo(os)
            os.flush()
        }

        // clean extraction folder
        extractionFolder.listFiles()?.forEach { it.deleteRecursively() }
        extractionFolder.mkdirs()

        // extract BAS -> extractionFolder
        if (!dynamo.extractBasFile(basFile.absolutePath, extractionFolder.absolutePath)) {
            ProLog.w(MODULE_NAME, "Unable to extract bas file: ${basFile.name}")
            return Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)
        }

        // find and parse manifest
        val manifestFile = extractionFolder.listFiles()
            ?.firstOrNull { it.name.contains("manifest", ignoreCase = true) }
            ?: return Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)

        val manifest = manifestFile.inputStream().use { ManifestParser().parse(it) }
        Outcome.Ok(manifest)

    } catch (t: Throwable) {
        ProLog.w(MODULE_NAME, "extractFirmwareFile failed: ${t.message}")
        Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)
    }
}




// In Bloc/ViewModel
override suspend fun evaluateFirmwareCompatibility(
    activeOSVersion: DynamoYoctoOS,
    firmwareFileInputStream: InputStream,                 // <— pass the stream in
): Outcome<DynamoFirmwareCompatibilityStatus> {

    update { s -> s.copy(firmwareCompatibilityStatus = DynamoFirmwareCompatibilityStatus.Evaluating) }

    return withContext(Dispatchers.IO) {
        // 1) Extract manifest from the (possibly decrypted) BAS
        val manifestOutcome = eFirmwareService.extractFirmwareFile(firmwareFileInputStream)

        // 2) Branch on extraction outcome
        when (manifestOutcome) {
            is Outcome.Error -> {
                Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)
            }
            is Outcome.Ok -> {
                val manifest = manifestOutcome.value
                // cache for UI/debug if you keep it in state
                update { s -> s.copy(releaseManifest = manifest) }

                // 3) Your existing compatibility logic (unchanged)
                eFirmwareService.evaluateFirmwareCompatibility(
                    bedStatusBloc.getSomOS(),
                    manifest
                )
            }
        }
    }
}

