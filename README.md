override suspend fun evaluateFirmwareCompatibility(
    activeOsVersion: DynamoYoctoOS,
    releaseManifest: SoftwareReleaseManifest
): Outcome<DynamoFirmwareCompatibilityStatus> {

    var containsOS = false
    var shouldSendOsUpdate = false

    // ✅ Single source of truth for OS version comparison
    shouldSendOsUpdate = shouldSendOsFromManifest(releaseManifest)

    // -------------------------------------------------
    // 1️⃣ Detect whether OS package exists
    // -------------------------------------------------
    extractionFolder.listFiles()?.forEach { file ->
        if (file.name == DynamoosTarGzFileName) {
            containsOS = true
            return@forEach
        }
    }

    // -------------------------------------------------
    // 2️⃣ Explicitly react only when dynamoAppTar exists
    //    (NO pkgOsVersion / NO manual SemVer)
    // -------------------------------------------------
    extractionFolder.listFiles()?.forEach { file ->
        if (file.name == DynamoosTarGzFileName) return@forEach

        if (file.name == dynamoAppTar) {
            // ✅ reuse existing logic only
            shouldSendOsUpdate = shouldSendOsFromManifest(releaseManifest)
            return@forEach
        }
    }

    // -------------------------------------------------
    // 3️⃣ OS not included at all
    // -------------------------------------------------
    if (!containsOS) {
        return Outcome.Ok(DynamoFirmwareCompatibilityStatus.OSUpdateNotIncluded)
    }

    // -------------------------------------------------
    // 4️⃣ Final decision
    // -------------------------------------------------
    return if (shouldSendOsUpdate) {
        ProLog.i(
            MODULE_NAME,
            msg = "OS differs -> OSUpdateIncluded ($DynamoosTarGzFileName)"
        )
        Outcome.Ok(DynamoFirmwareCompatibilityStatus.OSUpdateIncluded)
    } else {
        ProLog.i(
            MODULE_NAME,
            msg = "OS same -> OSUpdateNotIncluded (skip $DynamoosTarGzFileName)"
        )
        Outcome.Ok(DynamoFirmwareCompatibilityStatus.OSUpdateNotIncluded)
    }
}
