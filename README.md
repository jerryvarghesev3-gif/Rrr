hh



QStringList extracted;

// Try as ZIP first
extracted = JlCompress::extractDir(sourceFilePath, destinationFolderPath);
if (extracted.isEmpty()) {
    qDebug() << "[FW] ZIP extraction empty, trying TAR fallback...";
    QProcess::execute("/system/bin/sh", {"-c",
        QString("tar -xf %1 -C %2").arg(sourceFilePath).arg(destinationFolderPath)});
    QDir dir(destinationFolderPath);
    extracted = dir.entryList(QDir::Files | QDir::NoDotAndDotDot);
}

if (extracted.isEmpty()) {
    ProLog().e(MODULE_NAME,
               qPrintable(QString("Extraction failed for %1").arg(sourceFilePath)));
    return false;
}















import android.content.Context
import java.io.File
import java.io.FileInputStream
import java.io.FileOutputStream
import javax.crypto.Cipher
import javax.crypto.CipherInputStream
import javax.crypto.spec.GCMParameterSpec

private const val GCM_NONCE_LENGTH = 12
private const val GCM_TAG_LENGTH = 16
private const val ALGO = "AES/GCM/NoPadding"

fun decryptFileToTemp(context: Context, encFile: File, plainSuffix: String): File {
    // Create temporary output directory
    val tmpDir = File(context.filesDir, "tmp").apply { mkdirs() }

    // Output file path — remove .enc and append desired suffix
    val out = File(tmpDir, encFile.name.removeSuffix(".enc") + plainSuffix)

    FileInputStream(encFile).use { fin ->
        val iv = ByteArray(GCM_NONCE_LENGTH)
        require(fin.read(iv) == iv.size) { "IV missing in ${encFile.name}" }

        val cipher = Cipher.getInstance(ALGO)
        val spec = GCMParameterSpec(GCM_TAG_LENGTH * 8, iv)
        cipher.init(Cipher.DECRYPT_MODE, encryptionKey, spec)

        CipherInputStream(fin, cipher).use { cin ->
            FileOutputStream(out).use { fout ->
                val buf = ByteArray(64 * 1024)
                while (true) {
                    val n = cin.read(buf)
                    if (n <= 0) break  // EOF reached
                    fout.write(buf, 0, n)
                }
                fout.fd.sync() // Flush data to disk before closing
            }
        }
    }

    require(out.length() > 0L) { "Decrypted empty: ${out.path}" }
    return out
}









@Composable
fun FirmwareCompatibilityModals(
    state: DFirmwareScreenState,
    uiEvent: (FirmwareEvent) -> Unit,
    isBasic: Boolean
) {
    val status = state.firmwareCompatibilityStatus

    // show exactly one modal based on status priority
    when (status) {

        DFirmwareCompatibilityStatus.Evaluating -> {
            // Loading while we parse/extract
            LoadingModal(
                title = "Loading",
                text = "Evaluating firmware compatibility..."
            )
        }

        // Show transfer-method picker for both “included” and “not included” cases.
        // The service will push exactly the right files later.
        DFirmwareCompatibilityStatus.OSUpdateIncluded,
        DFirmwareCompatibilityStatus.OSUpdateNotIncluded -> {
            Modal(
                title = "Transfer Method",
                text  = "Choose your transfer method.",
                confirmButton = {
                    ModalButton(label = "Wi-Fi") {
                        uiEvent(FirmwareEvent.FwUpdateMethodSelect(FirmwareUpdateType.WIFI))
                    }
                },
                dismissButton = {
                    ModalSecondaryButton(label = "USB") {
                        uiEvent(FirmwareEvent.FwUpdateMethodSelect(FirmwareUpdateType.USB, isBasic))
                    }
                }
            )
        }

        DFirmwareCompatibilityStatus.OSUpdateRequired -> {
            Modal(
                title = "Update Required",
                text  = "Operating System update state required.",
                dismissButton = {
                    ModalButton(label = "Cancel") {
                        uiEvent(FirmwareEvent.FwIncompatibleConfirm)
                    }
                }
            )
        }

        DFirmwareCompatibilityStatus.UpdateIncompatible -> {
            Modal(
                title = "Error",
                text  = "Incompatible update.",
                confirmButton = {
                    ModalButton(label = "Okay") {
                        uiEvent(FirmwareEvent.FwIncompatibleConfirm)
                    }
                }
            )
        }

        DFirmwareCompatibilityStatus.Error -> {
            Modal(
                title = "Error",
                text  = "Error evaluating firmware compatibility.",
                confirmButton = {
                    ModalButton(label = "Okay") {
                        uiEvent(FirmwareEvent.FwIncompatibleConfirm)
                    }
                }
            )
        }

        else -> { /* No modal */ }
    }

    // Separate Wi-Fi helper modal (only if we are *already* on a Wi-Fi path)
    if (state.isWifiMethodModalVisible ||
        status == DFirmwareCompatibilityStatus.OSUpdateIncludedNoInternet
    ) {
        if (state.isMiniWifiProfileAlertVisible) {
            MiniWifiProfileModal(
                onOkClicked = { uiEvent(FirmwareEvent.ConfirmMiniWifiProfileAlert) }
            )
        } else {
            WifiNotConnectedModal(
                modalTitle = "Wi-Fi Method",
                onCancelHotspot = { uiEvent(FirmwareEvent.CancelHotspotClicked) },
                onPickWifi      = { uiEvent(FirmwareEvent.FwUpdateMethodSelect(FirmwareUpdateType.WIFI)) },
                onSetupHotspot  = { uiEvent(FirmwareEvent.SetupHotspotClicked) }
            )
        }
    }
}














@Composable
fun FirmwareCompatibilityModals(
    state: DynamoFirmwareScreenState,
    uiEvent: (event: FirmwareEvent) -> Unit,
    isBasic: Boolean
) {
    // 1) Loading while we evaluate
    if (state.firmwareCompatibilityStatus == DFirmwareCompatibilityStatus.Evaluating) {
        LoadingModal(
            title = "Loading",
            text  = "Evaluating firmware compatibility."
        )
    }

    // 2) OS update INCLUDED but we still need transport choice
    if (state.firmwareCompatibilityStatus == DFirmwareCompatibilityStatus.OSUpdateNotIncluded) {
        Modal(
            title = "Transfer Method",
            text  = "Choose your transfer method.",
            confirmButton = {
                ModalButton(label = "Wi-Fi") {
                    uiEvent(FirmwareEvent.FwUpdateMethodSelect(FirmwareUpdateType.WIFI))
                }
            },
            dismissButton = {
                ModalSecondaryButton(label = "USB") {
                    uiEvent(FirmwareEvent.FwUpdateMethodSelect(FirmwareUpdateType.USB, isBasic))
                }
            }
        )
    }

    // 3) Generic compatibility error
    if (state.firmwareCompatibilityStatus == DFirmwareCompatibilityStatus.Error) {
        Modal(
            title = "Error",
            text  = "Error evaluating firmware compatibility.",
            confirmButton = {
                ModalButton(label = "Okay") {
                    uiEvent(FirmwareEvent.FwIncompatibleConfirm)
                }
            }
        )
    }

    // 4) Update incompatible with current system
    if (state.firmwareCompatibilityStatus == DFirmwareCompatibilityStatus.UpdateIncompatible) {
        Modal(
            title = "Error",
            text  = "Incompatible update.",
            confirmButton = {
                ModalButton(label = "Okay") {
                    uiEvent(FirmwareEvent.FwIncompatibleConfirm)
                }
            }
        )
    }

    // 5) Wi-Fi path: either “mini profile” alert or the Wi-Fi method chooser
    if (state.isWifiMethodModalVisible ||
        state.firmwareCompatibilityStatus == DFirmwareCompatibilityStatus.OSUpdateIncludedNoInternet
    ) {
        if (state.isMiniWifiProfileAlertVisible) {
            MiniWifiProfileModal(
                onOKClicked = { uiEvent(FirmwareEvent.ConfirmMiniWifiProfileAlert) }
            )
        } else {
            WifiNotConnectedModal(
                modalTitle      = "Wi-Fi Method",
                onCancelHotspot = { uiEvent(FirmwareEvent.CancelHotspotClicked) },
                onPickWifi      = { uiEvent(FirmwareEvent.FwUpdateMethodSelect(FirmwareUpdateType.WIFI)) },
                onSetupHotspot  = { uiEvent(FirmwareEvent.SetupHotspotClicked) },
            )
        }
    }
}
