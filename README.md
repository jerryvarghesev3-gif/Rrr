ll




override suspend fun evaluateFirmwareCompatibility(
    firmwareFileName: String,
    firmwareFileInputStream: java.io.InputStream,
    somVersion: SemVer,
    isBasicMode: Boolean,
    boardsToUpdate: List<DynamoBoard>?
) {
    // your current body goes here…
    // e.g.:
    val releaseManifestOutcome = dynamoFirmwareService.extractFirmwareFile(firmwareFileInputStream)
    if (releaseManifestOutcome is Outcome.Error) {
        ProLog.e(MODULE_NAME, msg = "Error unzipping bas file.")
        update { s -> s.copy(firmwareCompatibilityStatus = DynamoFirmwareCompatibilityStatus.Error) }
        return
    }
    val releaseManifest = (releaseManifestOutcome as Outcome.Ok).value

    val result = dynamoFirmwareService.evaluateFirmwareCompatibility(
        bedStatusBloc.getSomOS(),   // or somVersion if your service expects it
        releaseManifest
    )

    when (result) {
        is Outcome.Ok   -> update { s -> s.copy(firmwareCompatibilityStatus = result.value) }
        is Outcome.Error-> update { s -> s.copy(firmwareCompatibilityStatus = DynamoFirmwareCompatibilityStatus.Error) }
    }
}
