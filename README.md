enum class DynamoYoctoOS(val yoctoPrjVer: DSemVer) {
    INVALID(DSemVer(0, 0, 0, 0)),

    //Dunfell is 1.0.0.0 (this is correct base)
    DUNFELL(DSemVer(1, 0, 0, 0));

    companion object {
        fun fromVersion(findVer: DSemVer): DynamoYoctoOS {
            // treat all-zero as invalid
            if (findVer.major == 0 && findVer.minor == 0 && findVer.patch == 0 && findVer.build == 0) {
                return INVALID
            }

            //match ALL 4 parts (important!)
            return values().firstOrNull { yocto ->
                yocto.yoctoPrjVer.major == findVer.major &&
                yocto.yoctoPrjVer.minor == findVer.minor &&
                yocto.yoctoPrjVer.patch == findVer.patch &&
                yocto.yoctoPrjVer.build == findVer.build
            } ?: INVALID
        }
    }
}






val somFwSemVer = state.boardStatuses[DynamoBoard.SYSTEM_ON_MODULE]?.firmwareVersion

val somFwDVer = somFwSemVer
    ?.let { DSemVer.fromString(it.toString()) }
    ?: DSemVer()   // 0.0.0.0 fallback

fwBloc.evaluateFirmwareCompatibility(
    firmwareFileName = selectedFile.name ?: "",
    firmwareFileInputStream = inputStream,
    somVersion = somFwDVer,
    isBasicMode = isBasic,
    boardsToUpdate = if (isBasic) null else state.advancedBoardList
)









override suspend fun evaluateFirmwareCompatibility(
    activeOsVersion: DynamoYoctoOS, // keep it only for logging/validation
    releaseManifest: SoftwareReleaseManifest,
): Outcome<DynamoFirmwareCompatibilityStatus> {

    val files = extractionFolder.listFiles().orEmpty()

    // OS file can be tar.gz OR tar.gz.enc
    val containsOsTar = files.any {
        it.name.equals("firmware_imx6_dynamo.tar.gz", ignoreCase = true) ||
        it.name.equals("firmware_imx6_dynamo.tar.gz.enc", ignoreCase = true)
    }

    ProLog.i(
        MODULE_NAME,
        "Evaluating FW compatibility: detectedYocto=$activeOsVersion, extractedFiles=${files.map { it.name }}"
    )
    ProLog.i(MODULE_NAME, "OS package presence check: containsOsTar=$containsOsTar")

    if (!containsOsTar) {
        ProLog.i(MODULE_NAME, "OS not included -> OSUpdateNotIncluded")
        return Outcome.Ok(DynamoFirmwareCompatibilityStatus.OSUpdateNotIncluded)
    }

    // ✅ single source of truth: manifest vs device OS diff
    val manifestDecision = shouldSendOsFromManifest(releaseManifest)

    ProLog.i(MODULE_NAME, "Manifest decision (manifest OS != device OS): $manifestDecision")

    return if (manifestDecision) {
        ProLog.i(MODULE_NAME, "OSUpdateIncluded -> OS WILL be sent")
        Outcome.Ok(DynamoFirmwareCompatibilityStatus.OSUpdateIncluded)
    } else {
        ProLog.i(MODULE_NAME, "OSUpdateNotIncluded -> OS will NOT be sent")
        Outcome.Ok(DynamoFirmwareCompatibilityStatus.OSUpdateNotIncluded)
    }
}
