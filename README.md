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













override suspend fun evaluateFirmwareCompatibility(
    activeOsVersion: DynamoYoctoOS,
    releaseManifest: SoftwareReleaseManifest
): Outcome<DynamoFirmwareCompatibilityStatus> {

    var containsOS = false
    var shouldSendOsUpdate = false

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
    // 2️⃣ Decide OS update ONLY when dynamoAppTar exists
    //    + DUNFELL check here (AS YOU ASKED)
    // -------------------------------------------------
    extractionFolder.listFiles()?.forEach { file ->
        if (file.name == DynamoosTarGzFileName) return@forEach

        if (file.name == dynamoAppTar) {

            // ✅ DUNFELL CHECK (MISSING EARLIER)
            if (activeOsVersion == DynamoYoctoOS.DUNFELL) {
                // ✅ reuse existing logic ONLY
                shouldSendOsUpdate = shouldSendOsFromManifest(releaseManifest)
            } else {
                shouldSendOsUpdate = false
            }

            return@forEach
        }
    }

    // -------------------------------------------------
    // 3️⃣ OS not included at all
    // -------------------------------------------------
    if (!containsOS) {
        return Outcome.Ok(
            DynamoFirmwareCompatibilityStatus.OSUpdateNotIncluded
        )
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
            msg = "OS same / not Dunfell -> OSUpdateNotIncluded (skip $DynamoosTarGzFileName)"
        )
        Outcome.Ok(DynamoFirmwareCompatibilityStatus.OSUpdateNotIncluded)
    }
}






override suspend fun evaluateFirmwareCompatibility(
    activeOsVersion: DynamoYoctoOS,
    releaseManifest: SoftwareReleaseManifest
): Outcome<DynamoFirmwareCompatibilityStatus> {

    val files = extractionFolder.listFiles().orEmpty()

    val osTar = files.firstOrNull { it.name.equals(DynamoosTarGzFileName, ignoreCase = true) }
    val hasDynamoAppTar = files.any { it.name.equals(dynamoAppTar, ignoreCase = true) }

    // 1) If OS tar not present -> OS not included
    if (osTar == null) {
        ProLog.i(MODULE_NAME, "OS tar not found -> OSUpdateNotIncluded ($DynamoosTarGzFileName)")
        return Outcome.Ok(DynamoFirmwareCompatibilityStatus.OSUpdateNotIncluded)
    }

    // 2) Decide ONLY when dynamoAppTar exists AND device is DUNFELL
    var shouldSendOsUpdate =
        (hasDynamoAppTar && activeOsVersion == DynamoYoctoOS.DUNFELL) &&
            shouldSendOsFromManifest(releaseManifest)

    // 3) If manifest says "no update", re-check using tar version extraction (OLD LOGIC)
    //    This brings back DynamoextractYoctoVersion behavior.
    if (!shouldSendOsUpdate && hasDynamoAppTar && activeOsVersion == DynamoYoctoOS.DUNFELL) {
        when (val tarVer = DynamoextractYoctoVersion(osTar)) {
            is Outcome.Ok -> {
                val pkgOs: SemVer? = SemVer.fromString(tarVer.value.trim())
                val deviceOs: SemVer? = SemVer.fromString(dynamo.getOperatingSystemVersion().trim())

                if (pkgOs != null && deviceOs != null) {
                    shouldSendOsUpdate = (pkgOs != deviceOs)
                    ProLog.i(
                        MODULE_NAME,
                        "Fallback tar check: pkgOs=$pkgOs deviceOs=$deviceOs shouldSendOsUpdate=$shouldSendOsUpdate"
                    )
                } else {
                    ProLog.w(MODULE_NAME, "Fallback tar check failed: SemVer parse null")
                }
            }
            is Outcome.Error -> {
                ProLog.w(MODULE_NAME, "Fallback tar extract failed: ${tarVer.error}")
            }
        }
    }

    ProLog.i(
        MODULE_NAME,
        "OS decision: hasOsTar=true, hasDynamoAppTar=$hasDynamoAppTar, " +
            "activeOsVersion=$activeOsVersion, shouldSendOsUpdate=$shouldSendOsUpdate"
    )

    return if (shouldSendOsUpdate) {
        ProLog.i(MODULE_NAME, "OS differs -> OSUpdateIncluded ($DynamoosTarGzFileName)")
        Outcome.Ok(DynamoFirmwareCompatibilityStatus.OSUpdateIncluded)
    } else {
        ProLog.i(MODULE_NAME, "OS not needed -> OSUpdateNotIncluded (skip $DynamoosTarGzFileName)")
        Outcome.Ok(DynamoFirmwareCompatibilityStatus.OSUpdateNotIncluded)
    }
}
