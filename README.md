ll



override suspend fun eFirmwareCompatibility(
    firmwareFileName: String,
    firmwareFileInputStream: InputStream,
    somVersion: SomVer,
    isBasicMode: Boolean,
    boardsToUpdate: List<DBoard>?
) {
    update { state ->
        state.copy(firmwareCompatibilityStatus = DFirmwareCompatibilityStatus.Evaluating)
    }

    try {
        val releaseManifestOutcome =
            dFirmwareService.extractFirmwareFile(firmwareFileInputStream)

        if (releaseManifestOutcome !is Outcome.OK) {
            ProLog.e(MODULE_NAME, "Error unzipping bas file.")
            update { state ->
                state.copy(firmwareCompatibilityStatus = DFirmwareCompatibilityStatus.Error)
            }
            return
        }

        update { state ->
            state.copy(releaseManifest = releaseManifestOutcome.value)
        }

        if (bedStatusBloc.state.boardStatuses[DBoard.SYSTEM_ON_MODULE]!!.connectStatus ==
            BoardConnectStatus.NOT_CONFIGURED
        ) {
            // SOM not configured -> only USB
            if (isBasicMode) {
                updateFirmware()
            } else {
                updateFirmware(boardsToUpdate)
            }

            update { state ->
                state.copy(firmwareCompatibilityStatus = DFirmwareCompatibilityStatus.Unchecked)
            }
        } else {
            val result = dFirmwareService.evaluateFirmwareCompatibility(
                bedStatusBloc.getSomOS(),
                releaseManifestOutcome.value
            )

            if (result is Outcome.OK) {

                if (isBasicMode) {
                    // BASIC / Wi-Fi MODE
                    update { state ->
                        state.copy(firmwareCompatibilityStatus = result.value)
                    }

                    val isNetworkSame = dWifiService.isBedConnectedToPhoneNetwork()

                    // ----- Wi-Fi OS update path -----
                    if (result.value == DFirmwareCompatibilityStatus.OsUpdateIncluded) {
                        if (isNetworkSame is Outcome.OK) {
                            // Wi-Fi OS transfer
                            transferFirmwareFiles()
                        } else {
                            // OS included but no internet
                            update { state ->
                                state.copy(
                                    firmwareCompatibilityStatus =
                                        DFirmwareCompatibilityStatus.OsUpdateIncludedNoInternet
                                )
                            }
                        }
                    }
                } else {
                    // NON-BASIC / USB MODE

                    // ***** CHANGED BLOCK *****
                    // Previously: if (result.value == OsUpdateNotIncluded)
                    // Now allow USB even when OS is INCLUDED
                    if (result.value == DFirmwareCompatibilityStatus.OsUpdateNotIncluded ||
                        result.value == DFirmwareCompatibilityStatus.OsUpdateIncluded
                    ) {
                        // In your UI, status Unchecked triggers the USB flow
                        update { state ->
                            state.copy(
                                firmwareCompatibilityStatus =
                                    DFirmwareCompatibilityStatus.Unchecked
                            )
                        }
                        updateFirmware(boardsToUpdate)
                    }
                    // ***** END OF CHANGED BLOCK *****

                    else if (
                        result.value == DFirmwareCompatibilityStatus.OsUpdateRequired ||
                        result.value == DFirmwareCompatibilityStatus.UpdateIncompatible
                    ) {
                        update { state ->
                            state.copy(firmwareCompatibilityStatus = result.value)
                        }

                        if (bedStatusBloc.getSomOS() == DYoctoOS.DUNFELL) {
                            update { state ->
                                state.copy(
                                    firmwareCompatibilityStatus =
                                        DFirmwareCompatibilityStatus.UpdateIncompatible
                                )
                            }
                        } else {
                            ProLog.e(
                                MODULE_NAME,
                                "SOM version = ${bedStatusBloc.getSomOS()}. " +
                                    "Bas file includes OS. Cannot push updateState."
                            )
                            update { state ->
                                state.copy(
                                    firmwareCompatibilityStatus =
                                        DFirmwareCompatibilityStatus.Error
                                )
                            }
                        }
                    }
                    // NOTE: old separate OsUpdateIncluded branch is no longer needed,
                    // because OsUpdateIncluded is now handled in the USB block above.
                }
            } else if (result is Outcome.Error) {
                ProLog.w(MODULE_NAME, "Firmware Compatibility failed.")
                update { state ->
                    state.copy(
                        firmwareCompatibilityStatus =
                            result.error as DFirmwareCompatibilityStatus
                    )
                }
            }
        }
    } catch (e: Exception) {
        ProLog.e(
            MODULE_NAME,
            "Firmware Compatibility exception = ${e.toString()} msg = ${e.message}."
        )
        update { state ->
            state.copy(firmwareCompatibilityStatus = DFirmwareCompatibilityStatus.Error)
        }
    }
}
