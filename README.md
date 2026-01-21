override suspend fun transferFirmwareFiles() {
    val includeOs =
        (state.pendingCompatibilityResult == DynamoFirmwareCompatibilityStatus.OSUpdateIncluded) ||
        (state.firmwareCompatibilityStatus == DynamoFirmwareCompatibilityStatus.OSUpdateIncluded)

    dynamoFirmwareService.sshFirmwareUpdate(
        state.bedStatusesAndConfig.boardStatuses[DynamoBoard.SYSTEM_ON_MODULE]?.firmwareVersion!!,
        state.releaseManifest,
        shouldSendOsUpdate = includeOs
    ).collect { firmwareUpdateStatus ->
        update { it.copy(firmwareUpdateStatus = firmwareUpdateStatus) }

        if (firmwareUpdateStatus is DynamoFirmwareUpdateStatus.Complete) {
            FirebaseCustomAnalytics.fwUpdateComplete(
                FirebaseCustomAnalytics.Device.DYNAMO,
                state.bedStatusesAndConfig.boardStatuses[DynamoBoard.MASTER_CONTROL_BOARD]?.firmwareVersion.toString(),
                state.bedStatusesAndConfig.bedConfig.serialNumber,
                FirebaseCustomAnalytics.FwUpdateType.WIFI,
                success = true
            )
        } else if (firmwareUpdateStatus is DynamoFirmwareUpdateStatus.Error.Process) {
            FirebaseCustomAnalytics.fwUpdateComplete(
                FirebaseCustomAnalytics.Device.DYNAMO,
                state.bedStatusesAndConfig.boardStatuses[DynamoBoard.MASTER_CONTROL_BOARD]?.firmwareVersion.toString(),
                state.bedStatusesAndConfig.bedConfig.serialNumber,
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
