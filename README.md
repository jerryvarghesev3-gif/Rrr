gg


override suspend fun evaluateFirmwareCompatibility(
    activeOsVersion: DYoctoOS,
    releaseManifest: SoftwareReleaseManifest
): Outcome<DFirmwareCompatibilityStatus> {

    // Only ensure the DOS image exists and is encrypted; HW will decrypt on the device.
    extractionFolder.listFiles()?.forEach { file ->
        if (file.name == DosTarGzFileName) {
            return if (file.name.endsWith(".enc", ignoreCase = true)) {
                Outcome.Ok(DFirmwareCompatibilityStatus.OSUpdateIncluded)
            } else {
                ProLog.e(MODULE_NAME, "Expected encrypted DOS (.enc), got ${file.name}")
                Outcome.Error(DFirmwareCompatibilityStatus.Error)
            }
        }
    }

    ProLog.e(MODULE_NAME, "DOS package ($DosTarGzFileName) not found in extraction folder.")
    return Outcome.Error(DFirmwareCompatibilityStatus.Error)
}





override suspend fun evaluateFirmwareCompatibility(
    firmwareFileName: String,
    firmwareFileInputStream: InputStream,
    somVersion: SemVer,
    isBasicMode: Boolean,
    boardsToUpdate: List<DBoard>?
)

update { state -> state.copy(firmwareCompatibilityStatus = DFirmwareCompatibilityStatus.Evaluating) }

// --- EARLY BYPASS FOR ENCRYPTED BAS ---
if (firmwareFileName.endsWith(".bas.enc", ignoreCase = true)) {
    // HW decrypts; we just proceed to push the same encrypted file.
    // Mark status to the "push" path your UI already expects.
    update { s -> s.copy(firmwareCompatibilityStatus = DFirmwareCompatibilityStatus.OSUpdateIncluded) }

    if (isBasicMode) {
        // Your existing basic-mode updater
        updateFirmware()
    } else {
        // Non-basic: only push if bed is on the phone AP (this mirrors your current logic)
        val isNetworkSame = dWifiService.isBedConnectedToPhoneNetwork()
        if (isNetworkSame is Outcome.Ok) {
            transferFirmwareFiles()        // <— this is your existing push
        } else {
            update { s ->
                s.copy(firmwareCompatibilityStatus = DFirmwareCompatibilityStatus.OSUpdateIncludedNoInternet)
            }
        }
    }
    return
}
// --- END EARLY BYPASS ---
