
val result = dynamoFirmwareService.evaluateFirmwareCompatibility(
    bedStatusBloc.getSomOS(),
    releaseManifestOutcome.value
)

println("2nd evaluate result = $result")
println("2nd evaluate somOS = ${bedStatusBloc.getSomOS()}")
println("2nd evaluate manifest = ${releaseManifestOutcome.value}")

if (result is Outcome.Ok) {

    // ✅ NEW: this is the only important wiring you were missing
    shouldSendOsUpdate = (result.value == DynamoFirmwareCompatibilityStatus.OSUpdateIncluded)

    if (isBasicMode) {
        // Keep whatever you already do for UI state
        update { state -> state.copy(firmwareCompatibilityStatus = result.value) }

        val isNetworkSame = dynamoWifiService.isBedConnectedToPhoneNetwork()

        // ✅ If OS included AND network ok → transfer (OS + others)
        // ✅ If OS NOT included → transfer other binaries (transferFirmwareFiles will skip OS because shouldSendOsUpdate=false)
        if (isNetworkSame is Outcome.Ok) {
            transferFirmwareFiles()
        } else if (shouldSendOsUpdate) {
            // OS needed but can't use Wi-Fi
            update { state ->
                state.copy(
                    firmwareCompatibilityStatus =
                        DynamoFirmwareCompatibilityStatus.OSUpdateIncludedNoInternet
                )
            }
        } else {
            // No OS needed; Wi-Fi not same. Keep old behavior (do nothing / let user pick USB / show status if you already have one)
            // If you already have a status for this case, set it here. Otherwise leave it.
        }

        return
    }

    // ✅ ADVANCED: keep your existing logic exactly same
    if (result.value == DynamoFirmwareCompatibilityStatus.OSUpdateNotIncluded) {
        update { state ->
            state.copy(firmwareCompatibilityStatus = DynamoFirmwareCompatibilityStatus.Unchecked)
        }
        updateFirmware(boardsToUpdate)
        return
    } else if (
        result.value == DynamoFirmwareCompatibilityStatus.OSUpdateRequired ||
        result.value == DynamoFirmwareCompatibilityStatus.UpdateIncompatible
    ) {
        update { state -> state.copy(firmwareCompatibilityStatus = result.value) }
        return
    } else if (result.value == DynamoFirmwareCompatibilityStatus.OSUpdateIncluded) {
        if (bedStatusBloc.getSomOS() == DynamoYoctoOS.DUNFELL) {
            update { state ->
                state.copy(firmwareCompatibilityStatus = DynamoFirmwareCompatibilityStatus.UpdateIncompatible)
            }
        } else {
            ProLog.e(MODULE_NAME, "SOM version = ${bedStatusBloc.getSomOS()}. Bas file includes OS. Cannot push updateState.")
            update { state ->
                state.copy(firmwareCompatibilityStatus = DynamoFirmwareCompatibilityStatus.Error)
            }
        }
        return
    }

    // fallback
    update { state -> state.copy(firmwareCompatibilityStatus = result.value) }
    return
}

if (result is Outcome.Error) {
    ProLog.w(MODULE_NAME, "Firmware Compatibility failed.")
    update { state ->
        state.copy(firmwareCompatibilityStatus = result.error as DynamoFirmwareCompatibilityStatus)
    }
    return
}
