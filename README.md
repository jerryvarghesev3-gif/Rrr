# Rrr
Drd



override suspend fun evaluateFirmwareCompatibility(
    activeOsVersion: DYoctoOS,
    releaseManifest: SoftwareReleaseManifest
): Outcome<DFirmwareCompatibilityStatus> {

    // Look only for the DOS image and ensure it's .enc
    extractionFolder.listFiles()?.forEach { file ->
        if (file.name == DosTarGzFileName) {
            return if (file.name.endsWith(".enc", ignoreCase = true)) {
                // ✅ We found the expected encrypted DOS image; let the flow continue to push.
                Outcome.Ok(DFirmwareCompatibilityStatus.OSUpdateNotIncluded)
                // (Keep the same success enum your callers already expect)
            } else {
                // Found DOS file but it isn’t encrypted — treat as incompatible
                ProLog.e(MODULE_NAME, "Expected encrypted DOS image (.enc), got: ${file.name}")
                Outcome.Error(DFirmwareCompatibilityStatus.Error)
            }
        }
    }

    // No DOS package found at all
    ProLog.e(MODULE_NAME, "DOS package ($DosTarGzFileName) not found in extraction folder.")
    return Outcome.Error(DFirmwareCompatibilityStatus.Error)
}
