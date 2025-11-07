ll


override suspend fun extractFirmwareFile(
    firmwareFileInputStream: InputStream
): Outcome<SoftwareReleaseManifest> {

    val basFile = File("${context.filesDir.absolutePath}/firmware_raw.bas")
    val os = basFile.outputStream()

    return try {
        firmwareFileInputStream.copyTo(os)
        os.flush()

        // clean extraction folder
        extractionFolder.listFiles()?.forEach { it.deleteRecursively() }
        extractionFolder.mkdirs()

        if (!dynamo.extractBasFile(basFile.absolutePath, extractionFolder.absolutePath)) {
            ProLog.w(MODULE_NAME, "Unable to extract bas file: ${basFile.name}")
            Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)
        } else {
            // Must have a manifest; if not, error out right here
            val manifestFile = extractionFolder
                .listFiles()
                ?.firstOrNull { it.name.contains("manifest", ignoreCase = true) }
                ?: return Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)

            // Make sure stream is closed even on parse failure
            val releaseManifest = try {
                manifestFile.inputStream().use { ManifestParser().parse(it) }
            } catch (e: Exception) {
                ProLog.w(MODULE_NAME, "Unable to parse manifest file.")
                return Outcome.Error(e)
            }

            Outcome.Ok(releaseManifest)
        }
    } finally {
        try { os.close() } catch (_: Throwable) {}
    }
}



override suspend fun evaluateFirmwareCompatibility(
    firmwareFileName: String,
    firmwareFileInputStream: InputStream,
    somVersion: SemVer,
    isBasicMode: Boolean,
    boardsToUpdate: List<DBoard>?
) {
    // enter loading
    update { state -> state.copy(firmwareCompatibilityStatus = DFirmwareCompatibilityStatus.Evaluating) }

    val result = withContext(Dispatchers.IO) {
        withTimeoutOrNull(30_000) {   // 30s hard ceiling
            // --- existing code path, untouched except for early returns ---
            val releaseManifestOutcome = eFirmwareService.extractFirmwareFile(firmwareFileInputStream)
            if (releaseManifestOutcome is Outcome.Error) return@withTimeoutOrNull releaseManifestOutcome

            val releaseManifest = (releaseManifestOutcome as Outcome.Ok).value
            update { s -> s.copy(releaseManifest = releaseManifest) }

            // your existing compatibility logic…
            eFirmwareService.evaluateFirmwareCompatibility(bedStatusBloc.getSomOS(), releaseManifest)
        }
    }

    when (result) {
        is Outcome.Ok -> {
            // your existing branching that may call transferFirmwareFiles(), etc.
            update { s -> s.copy(firmwareCompatibilityStatus = result.value) }
        }
        is Outcome.Error -> {
            ProLog.e(MODULE_NAME, "Firmware Compatibility failed.")
            update { s -> s.copy(firmwareCompatibilityStatus = DFirmwareCompatibilityStatus.Error) }
        }
        null -> { // timeout
            ProLog.e(MODULE_NAME, "Firmware compatibility timed out.")
            update { s -> s.copy(firmwareCompatibilityStatus = DFirmwareCompatibilityStatus.Error) }
        }
    }
}


