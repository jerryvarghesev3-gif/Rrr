kk



override suspend fun extractFirmwareFile(
    firmwareFileInputStream: InputStream
): Outcome<SoftwareReleaseManifest> = withContext(Dispatchers.IO) {
    val basFile = File("${context.filesDir.absolutePath}/firmware.bas")

    // 1) Write the incoming stream to a .bas temp file
    basFile.outputStream().use { out ->
        firmwareFileInputStream.copyTo(out)
        out.flush()
    }

    // 2) Clean extraction folder
    extractionFolder.listFiles()?.forEach { it.deleteRecursively() }
    extractionFolder.mkdirs()

    // 3) Extract BAS (no decryption of internal .enc members)
    val extractedOk = try {
        // replace with your real extractor:
        // centralla.extractBasFile(basFile.absolutePath, extractionFolder.absolutePath)
        dynamo.extractBasFile(basFile.absolutePath, extractionFolder.absolutePath)
    } catch (_: Throwable) {
        false
    }

    if (!extractedOk) {
        ProLog.w(MODULE_NAME, msg = "Unable to extract bas file: ${basFile.name}")
        return@withContext Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)
    }

    // 4) Find manifest (now plain text after extraction step)
    val manifestFile = extractionFolder
        .listFiles()
        ?.firstOrNull { it.name.contains(manifestMarker, ignoreCase = true) }
        ?: return@withContext Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)

    // 5) Parse manifest
    val releaseManifest = try {
        manifestFile.inputStream().use { ManifestParser().parse(it) }
    } catch (e: Exception) {
        ProLog.w(MODULE_NAME, msg = "Unable to parse manifest file.")
        return@withContext Outcome.Error(e)
    }

    Outcome.Ok(releaseManifest)
}





override suspend fun evaluateFirmwareCompatibility(
    activeOSVersion: DynamoYoctoOS,
    releaseManifest: SoftwareReleaseManifest
): Outcome<DynamoFirmwareCompatibilityStatus> = withContext(Dispatchers.IO) {
    // We only validate presence of an OS package inside the extracted BAS.
    val files = extractionFolder.listFiles().orEmpty()

    // Accept either naming convention your BAS may carry.
    val hasOsPkg = files.any { f ->
        val n = f.name.lowercase()
        n == dynamoOsTarGzFileName.lowercase() || n == ospkgTarGzFileName.lowercase()
    }

    return@withContext if (hasOsPkg) {
        Outcome.Ok(DynamoFirmwareCompatibilityStatus.OSUpdateIncluded)
    } else {
        Outcome.Ok(DynamoFirmwareCompatibilityStatus.OSUpdateNotIncluded)
    }
}






override suspend fun evaluateFirmwareCompatibility(
    activeOSVersion: DynamoYoctoOS,
    releaseManifest: SoftwareReleaseManifest
): Outcome<DynamoFirmwareCompatibilityStatus> = withContext(Dispatchers.IO) {
    // We only validate presence of an OS package inside the extracted BAS.
    val files = extractionFolder.listFiles().orEmpty()

    // Accept either naming convention your BAS may carry.
    val hasOsPkg = files.any { f ->
        val n = f.name.lowercase()
        n == dynamoOsTarGzFileName.lowercase() || n == ospkgTarGzFileName.lowercase()
    }

    return@withContext if (hasOsPkg) {
        Outcome.Ok(DynamoFirmwareCompatibilityStatus.OSUpdateIncluded)
    } else {
        Outcome.Ok(DynamoFirmwareCompatibilityStatus.OSUpdateNotIncluded)
    }
}




override suspend fun evaluateFirmwareCompatibility(
    firmwareFileName: String,
    firmwareFileInputStream: InputStream,
    somVersion: SemVer,
    isBasMode: Boolean,
    boardsToUpdate: List<DynamoBoard>
) {
    // show "Evaluating" UI state as you already do…

    // 1) Extract BAS and get manifest
    val releaseManifestOutcome = dynamoFirmwareService.extractFirmwareFile(firmwareFileInputStream)
    if (releaseManifestOutcome is Outcome.Error) {
        ProLog.e(MODULE_NAME, "Error unzipping bas file.")
        update { s -> s.copy(firmwareCompatibilityStatus = DynamoFirmwareCompatibilityStatus.Error) }
        return
    }
    val releaseManifest = (releaseManifestOutcome as Outcome.Ok).value

    // 2) Ask service to validate OS presence only
    val result = dynamoFirmwareService.evaluateFirmwareCompatibility(
        bedStatusBloc.getSomOS(),   // or your active OS source
        releaseManifest
    )

    if (result is Outcome.Ok) {
        val status = result.value
        // OS must be included; otherwise fail here
        if (status == DynamoFirmwareCompatibilityStatus.OSUpdateIncluded) {
            update { s -> s.copy(firmwareCompatibilityStatus = status) }
            // (From here your existing flow can decide to allow "update" button, etc.)
        } else {
            update { s -> s.copy(firmwareCompatibilityStatus = DynamoFirmwareCompatibilityStatus.OSUpdateNotIncluded) }
        }
    } else {
        update { s -> s.copy(firmwareCompatibilityStatus = DynamoFirmwareCompatibilityStatus.Error) }
    }
}




