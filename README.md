override suspend fun evaluateFirmwareCompatibility(
    firmwareFileName: String,
    firmwareFileInputStream: InputStream,
    somVersion: SemVer,
    isBasicMode: Boolean,
    boardsToUpdate: List<DynamoBoard>?
) {
    update { state ->
        state.copy(firmwareCompatibilityStatus = DynamoFirmwareCompatibilityStatus.Evaluating)
    }

    try {
        val releaseManifestOutcome =
            dynamoFirmwareService.extractFirmwareFile(firmwareFileInputStream)

        if (releaseManifestOutcome !is Outcome.Ok) {
            ProLog.e(MODULE_NAME, "Error unzipping bas file.")
            update { state ->
                state.copy(firmwareCompatibilityStatus = DynamoFirmwareCompatibilityStatus.Error)
            }
            return
        }

        update { state ->
            state.copy(releaseManifest = releaseManifestOutcome.value)
        }

        // ---------------------------
        // KEEP YOUR OLD NOT_CONFIGURED
        // ---------------------------
        if (bedStatusBloc.state.boardStatuses[DynamoBoard.SYSTEM_ON_MODULE]!!.connectStatus ==
            BoardConnectStatus.NOT_CONFIGURED
        ) {
            if (isBasicMode) {
                // ✅ NEW REQUIREMENT: in BASIC don’t auto update, show popup always
                update { state ->
                    state.copy(
                        firmwareCompatibilityStatus = DynamoFirmwareCompatibilityStatus.Unchecked,
                        pendingCompatibilityResult = DynamoFirmwareCompatibilityStatus.OSUpdateNotIncluded
                    )
                }
                return
            } else {
                // ✅ Advanced unchanged
                updateFirmware(boardsToUpdate)
                return
            }
        }

        val result = dynamoFirmwareService.evaluateFirmwareCompatibility(
            bedStatusBloc.getSomOs(),
            releaseManifestOutcome.value
        )

        if (result is Outcome.Ok) {

            // ---------------------------
            // ADVANCED MODE = unchanged
            // ---------------------------
            if (!isBasicMode) {
                update { state -> state.copy(firmwareCompatibilityStatus = result.value) }
                updateFirmware(boardsToUpdate)
                return
            }

            // ---------------------------
            // BASIC MODE (KEEP YOUR OLD LOGIC)
            // ---------------------------

            // Keep your old behavior: update state to result.value first
            update { state -> state.copy(firmwareCompatibilityStatus = result.value) }

            // Keep your old network + transfer logic exactly
            val isNetworkSame = dynamoWifiService.isBedConnectedToPhoneNetwork()
            if (result.value == DynamoFirmwareCompatibilityStatus.OSUpdateIncluded && isNetworkSame is Outcome.Ok) {
                transferFirmwareFiles()
            } else if (result.value == DynamoFirmwareCompatibilityStatus.OSUpdateIncluded) {
                update {
                    it.copy(firmwareCompatibilityStatus = DynamoFirmwareCompatibilityStatus.OSUpdateIncludedNoInternet)
                }
            }

            // ✅ NEW REQUIREMENT (ONLY ADDITION):
            // Always show popup afterwards (for BOTH cases: OS included OR not included)
            update { state ->
                state.copy(
                    firmwareCompatibilityStatus = DynamoFirmwareCompatibilityStatus.Unchecked, // popup trigger
                    pendingCompatibilityResult = result.value // store OSIncluded/NotIncluded for popup
                )
            }
            return
        }

        if (result is Outcome.Error) {
            ProLog.w(MODULE_NAME, "Firmware Compatibility failed.")
            update { state ->
                state.copy(firmwareCompatibilityStatus = result.error as DynamoFirmwareCompatibilityStatus)
            }
            return
        }

    } catch (e: Exception) {
        ProLog.e(MODULE_NAME, "Firmware Compatibility exception = $e msg = ${e.message}")
        update { state ->
            state.copy(firmwareCompatibilityStatus = DynamoFirmwareCompatibilityStatus.Error)
        }
    }
}
