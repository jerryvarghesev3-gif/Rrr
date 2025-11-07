ll



override suspend fun evaluateFirmwareCompatibility(
    firmwareFileName: String,
    firmwareFileInputStream: InputStream,
    somVersion: SemVer,
    isBasicMode: Boolean,
    boardsToUpdate: List<DBoard>?
) {
    // Enter "loading"
    update { s -> s.copy(firmwareCompatibilityStatus = DFirmwareCompatibilityStatus.Evaluating) }

    try {
        val result: Outcome<DFirmwareCompatibilityStatus> =
            withContext(Dispatchers.IO) {
                // Hard ceiling to prevent endless spinner
                withTimeoutOrNull(30_000L) {
                    // 1) Unpack BAS and parse manifest (no decryption of *.enc here)
                    when (val mf = firmwareService.extractFirmwareFile(firmwareFileInputStream)) {
                        is Outcome.Ok -> {
                            val manifest = mf.value
                            // cache for UI/debug (you already render this in modals)
                            update { s -> s.copy(releaseManifest = manifest) }

                            // 2) Your existing logic decides the final status
                            firmwareService.evaluateFirmwareCompatibility(
                                bedStatusBloc.getSomOS(),
                                manifest
                            )
                        }
                        is Outcome.Error -> {
                            @Suppress("UNCHECKED_CAST")
                            mf as Outcome<DFirmwareCompatibilityStatus>
                        }
                    }
                } ?: Outcome.Error(DFirmwareCompatibilityStatus.Error) // timeout -> generic error
            }

        // Reflect outcome
        when (result) {
            is Outcome.Ok -> update { s -> s.copy(firmwareCompatibilityStatus = result.value) }
            is Outcome.Error -> {
                val st = (result.error as? DFirmwareCompatibilityStatus)
                    ?: DFirmwareCompatibilityStatus.Error
                update { s -> s.copy(firmwareCompatibilityStatus = st) }
            }
        }
    } catch (e: Exception) {
        ProLog.e(MODULE_NAME, "Firmware Compatibility exception = ${e}")
        update { s -> s.copy(firmwareCompatibilityStatus = DFirmwareCompatibilityStatus.Error) }
    }
}




override suspend fun extractFirmwareFile(
    firmwareFileInputStream: InputStream
): Outcome<SoftwareReleaseManifest> {

    // Persist the uploaded BAS to app storage
    val basFile = File("${context.filesDir.absolutePath}/firmware_raw.bas")
    val extractionFolder = File("${context.filesDir.absolutePath}/fw/unpacked")

    // Ensure clean extraction dir
    if (extractionFolder.exists()) {
        extractionFolder.listFiles()?.forEach { it.deleteRecursively() }
        extractionFolder.deleteRecursively()
    }
    extractionFolder.mkdirs()

    // Copy stream -> file
    basFile.outputStream().use { os ->
        firmwareFileInputStream.copyTo(os)
        os.flush()
        (os.fd).sync()     // keep this for durability on some devices
    }

    // Ask your native/helper to extract the BAS (ZIP first, TAR fallback is inside)
    val ok = dynamo.extractBasFile(
        basFile.absolutePath,
        extractionFolder.absolutePath
    )
    if (!ok) {
        ProLog.w(MODULE_NAME, "Unable to extract bas file: ${basFile.name}")
        return Outcome.Error(DFirmwareCompatibilityStatus.Error)
    }

    // Find manifest inside the extracted folder
    val manifestFile = extractionFolder
        .walkTopDown()
        .firstOrNull { it.isFile && it.name.equals("manifest.xml", ignoreCase = true) }
        ?: return Outcome.Error(DFirmwareCompatibilityStatus.Error)

    return try {
        val releaseManifest = ManifestParser().parse(manifestFile.inputStream())
        Outcome.Ok(releaseManifest)
    } catch (e: Exception) {
        ProLog.w(MODULE_NAME, "Unable to parse manifest file. ${e.message}")
        Outcome.Error(DFirmwareCompatibilityStatus.Error)
    }
}



