
package com.hillrom.servicetool.usb

import android.content.BroadcastReceiver
import android.content.Context
import android.content.Intent
import android.hardware.usb.UsbDevice
import android.hardware.usb.UsbManager

class UsbPermissionReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        if (intent.action != UsbConst.ACTION_USB_PERMISSION) return

        val device = intent.getParcelableExtra<UsbDevice>(UsbManager.EXTRA_DEVICE)
        val granted = intent.getBooleanExtra(UsbManager.EXTRA_PERMISSION_GRANTED, false)

        // Store result somewhere simple (SharedPreferences) so ViewModel can read it
        val prefs = context.getSharedPreferences("usb_perm", Context.MODE_PRIVATE)
        prefs.edit()
            .putBoolean("granted", granted)
            .putString("deviceName", device?.deviceName ?: "")
            .apply()
    }
}









package com.hillrom.servicetool.usb

object UsbConst {
    const val ACTION_USB_PERMISSION = "com.hillrom.servicetool.USB_PERMISSION"
}











import android.app.PendingIntent
import android.content.Context
import android.content.Intent
import android.hardware.usb.UsbDevice
import android.hardware.usb.UsbManager
import com.hillrom.servicetool.usb.UsbConst

private fun requestUsbPermission(context: Context, device: UsbDevice) {
    val usbManager = context.getSystemService(Context.USB_SERVICE) as UsbManager

    val pi = PendingIntent.getBroadcast(
        context,
        0,
        Intent(UsbConst.ACTION_USB_PERMISSION),
        PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE
    )

    usbManager.requestPermission(device, pi)
}

private fun getFirstUsbDevice(context: Context): UsbDevice? {
    val usbManager = context.getSystemService(Context.USB_SERVICE) as UsbManager
    return usbManager.deviceList.values.firstOrNull()
}








private fun load(appContext: Context) {
    launch {
        // 1) connect first (safe)
        if (!connectionBloc.isConnected()) {
            updateState { it.copy(loadingStatus = LoadingStatus.Connecting) }

            if (connectionBloc.connect(Dynamo_GN2_Address.SYS_DIAG) is Outcome.Error) {
                updateState { it.copy(loadingStatus = LoadingStatus.Error) }
                return@launch
            }
        }

        // 2) request permission (this triggers popup IF not already granted)
        val dev = getFirstUsbDevice(appContext)
        if (dev == null) {
            // no usb device attached
            updateState {
                it.copy(
                    loadingStatus = LoadingStatus.Error,
                    incompatibleProductDialogVisible = true,
                    incompatibleProductMessage = "No USB device detected."
                )
            }
            return@launch
        }

        val usbManager = appContext.getSystemService(Context.USB_SERVICE) as UsbManager
        if (!usbManager.hasPermission(dev)) {
            requestUsbPermission(appContext, dev)

            // stop here; user must allow/deny. retry button can call load(appContext) again.
            updateState {
                it.copy(
                    loadingStatus = LoadingStatus.Error,
                    incompatibleProductDialogVisible = true,
                    incompatibleProductMessage = "USB permission required. Please tap Allow."
                )
            }
            return@launch
        }

        // 3) NOW it is safe to read dev strings and block wrong product BEFORE JNI
        val text = "${dev.productName} ${dev.manufacturerName}".uppercase()
        val detected = when {
            text.contains("P80") -> MedDevices.DYNAMO
            text.contains("P79") -> MedDevices.CENTRELLA
            else -> null
        }

        if (detected != null && detected != MedDevices.DYNAMO) {
            updateState {
                it.copy(
                    loadingStatus = LoadingStatus.Error,
                    incompatibleProductDialogVisible = true,
                    incompatibleProductMessage = "Invalid product connected. Please connect Dynamo."
                )
            }
            // stop any polling/collection BEFORE disconnect
            deviceBloc.stopBoardCollection()
            deviceBloc.stopBedStatusPoll()
            connectionBloc.disconnect()
            return@launch
        }

        // 4) only after passing -> start these (these touch JNI later)
        deviceBloc.startBoardCollection()
        loadProperties()          // your existing one
        deviceBloc.startBedStatusPoll()
    }
}









