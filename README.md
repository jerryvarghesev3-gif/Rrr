ll



import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.withContext
import kotlinx.coroutines.withTimeoutOrNull

override suspend fun evaluateFirmwareCompatibility(
    firmwareFileName: String,
    firmwareFileInputStream: InputStream,
    somVersion: SemVer,
    isBasicMode: Boolean,
    boardsToUpdate: List<DBoard>?
) {
    // 0) enter loading
    update { state ->
        state.copy(firmwareCompatibilityStatus = DFirmwareCompatibilityStatus.Evaluating)
    }

    // 1) Run extraction + evaluation off the main thread, with a hard ceiling
    val result: Outcome<DFirmwareCompatibilityStatus> = withContext(Dispatchers.IO) {
        withTimeoutOrNull(30_000L) {
            // --- Extract manifest from the (possibly decrypted) BAS ---
            when (val mf = eFirmwareService.extractFirmwareFile(firmwareFileInputStream)) {
                is Outcome.Ok -> {
                    val manifest = mf.value
                    // cache manifest for UI/debug
                    update { s -> s.copy(releaseManifest = manifest) }

                    // --- Your existing compatibility logic (unchanged) ---
                    // IMPORTANT: this must return Outcome<DFirmwareCompatibilityStatus>
                    eFirmwareService.evaluateFirmwareCompatibility(
                        bedStatusBloc.getSomOS(),
                        manifest
                    )
                }
                is Outcome.Error -> {
                    // normalize type for this withTimeoutOrNull{} result
                    Outcome.Error(DFirmwareCompatibilityStatus.Error)
                }
            }
        } ?: Outcome.Error(DFirmwareCompatibilityStatus.Error) // timeout -> error
    }

    // 2) Leave loading with a definitive, UI-friendly state
    when (result) {
        is Outcome.Ok -> {
            update { s -> s.copy(firmwareCompatibilityStatus = result.value) }
        }
        is Outcome.Error -> {
            ProLog.e(MODULE_NAME, "Firmware compatibility failed or timed out.")
            update { s -> s.copy(firmwareCompatibilityStatus = DFirmwareCompatibilityStatus.Error) }
        }
    }
}
