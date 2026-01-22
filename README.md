override suspend fun transferFirmwareFiles() {
    val somSemVer =
        state.bedStatusAndConfig.boardStatuses[DynamoBoard.SYSTEM_ON_MODULE]?.firmwareVersion

    //Convert SemVer (3-part) -> DSemVerpart)
    val somDSemVer = somSemVer?.let { sv ->
        DSemVer(
            major = sv.major,
            minor = sv.minor,
            patch = sv.patch,
            build = 0 // SemVer has no build; keep 0
        )
    } ?: DSemVer.Invalid

    dynamoFirmwareService.sshFirmwareUpdate(
        somVersion = somDSemVer,
        releaseManifest = state.releaseManifest,
        shouldSendOsUpdate = (state.firmwareCompatibilityStatus == DynamoFirmwareCompatibilityStatus.OSUpdateIncluded)
    )
        .collect { firmwareUpdateStatus ->
            update { state ->
                state.copy(firmwareUpdateStatus = firmwareUpdateStatus)
            }

            // log to analytics fw updateState completed successfully
            if (firmwareUpdateStatus is DynamoFirmwareUpdateStatus.Complete) {
                FirebaseCustomAnalytics.fwUpdateComplete(
                    FirebaseCustomAnalytics.Device.DYNAMO,
                    state.bedStatusAndConfig.boardStatuses[DynamoBoard.MASTER_CONTROL_BOARD]?.firmwareVersion.toString(),
                    state.bedStatusAndConfig.bedConfig.serialNumber,
                    FirebaseCustomAnalytics.FwUpdateType.WIFI,
                    success = true
                )
            }
            // log to analytics fw updateState failed
            else if (firmwareUpdateStatus is DynamoFirmwareUpdateStatus.Error.Process) {
                FirebaseCustomAnalytics.fwUpdateComplete(
                    FirebaseCustomAnalytics.Device.DYNAMO,
                    state.bedStatusAndConfig.boardStatuses[DynamoBoard.MASTER_CONTROL_BOARD]?.firmwareVersion.toString(),
                    state.bedStatusAndConfig.bedConfig.serialNumber,
                    FirebaseCustomAnalytics.FwUpdateType.WIFI,
                    success = false,
                    firmwareUpdateStatus.message()
                )
            }
        }

    if (wifiBloc.getHotspotIndex() != null) {
        wifiBloc.removeHotspot()
    }
}
