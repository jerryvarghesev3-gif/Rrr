ll





package com.hillrom.servicetool.usb

import android.app.PendingIntent
import android.content.BroadcastReceiver
import android.content.Context
import android.content.Intent
import android.content.IntentFilter
import android.hardware.usb.UsbDevice
import android.hardware.usb.UsbManager
import kotlinx.coroutines.CancellableContinuation
import kotlinx.coroutines.suspendCancellableCoroutine
import kotlin.coroutines.resume

object UsbProductGate {

    private const val ACTION_USB_PERMISSION = "com.hillrom.servicetool.USB_PERMISSION"

    /** Detects device type from USB *strings* (safe, no JNI). */
    fun detectFromUsbStrings(dev: UsbDevice?): MedDevices? {
        if (dev == null) return null

        val text = buildString {
            append(dev.productName ?: "")
            append(" ")
            append(dev.manufacturerName ?: "")
            append(" ")
            append(dev.deviceName ?: "")
        }.uppercase()

        // You told: Centrella = P7900..., Dynamo = P800...
        // safest: match P79* as Centrella, P80* as Dynamo.
        return when {
            text.contains("P79") -> MedDevices.CENTRELLA
            text.contains("P80") -> MedDevices.DYNAMO
            else -> null
        }
    }

    fun findFirstUsbDevice(context: Context): UsbDevice? {
        val usb = context.getSystemService(Context.USB_SERVICE) as UsbManager
        return usb.deviceList.values.firstOrNull()
    }

    /** Shows system USB permission popup and waits for user choice. */
    suspend fun requestUsbPermission(context: Context, dev: UsbDevice): Boolean {
        val appContext = context.applicationContext
        val usb = appContext.getSystemService(Context.USB_SERVICE) as UsbManager

        if (usb.hasPermission(dev)) return true

        val pi = PendingIntent.getBroadcast(
            appContext,
            0,
            Intent(ACTION_USB_PERMISSION),
            PendingIntent.FLAG_IMMUTABLE
        )

        return suspendCancellableCoroutine { cont: CancellableContinuation<Boolean> ->
            val receiver = object : BroadcastReceiver() {
                override fun onReceive(ctx: Context?, intent: Intent?) {
                    if (intent?.action != ACTION_USB_PERMISSION) return

                    val granted = intent.getBooleanExtra(UsbManager.EXTRA_PERMISSION_GRANTED, false)
                    runCatching { appContext.unregisterReceiver(this) }
                    if (cont.isActive) cont.resume(granted)
                }
            }

            appContext.registerReceiver(receiver, IntentFilter(ACTION_USB_PERMISSION))
            cont.invokeOnCancellation { runCatching { appContext.unregisterReceiver(receiver) } }

            usb.requestPermission(dev, pi)
        }
    }

    /**
     * Full safe gate:
     * 1) find usb device
     * 2) request permission (shows popup)
     * 3) detect product from usb strings
     */
    suspend fun detectDeviceViaUsbPermission(context: Context): MedDevices? {
        val dev = findFirstUsbDevice(context) ?: return null
        val ok = requestUsbPermission(context, dev)
        if (!ok) return null
        return detectFromUsbStrings(dev)
    }
}

/** Match your real enum name/package */
enum class MedDevices { CENTRELLA, DYNAMO }




private suspend fun blockIfWrongProduct_Centrella(appContext: Context): Boolean {
    val detected = com.hillrom.servicetool.usb.UsbProductGate.detectDeviceViaUsbPermission(appContext)

    // If we cannot detect -> allow (or you can choose to block). I recommend allow.
    if (detected == null) return false

    if (detected != MedDevices.CENTRELLA) {
        updateState {
            it.copy(
                loadingStatus = LoadingStatus.Error,
                incompatibleProductDialogVisible = true,
                incompatibleProductMessage = "Invalid product connected. Please connect the correct device."
            )
        }

        // stop anything that might trigger JNI
        deviceBloc.stopBoardCollection()
        deviceBloc.stopBedStatusPoll()
        connectionBloc.disconnect()

        return true // BLOCKED
    }
    return false // OK
}







private fun load(appContext: Context) {
    launch {
        if (!connectionBloc.isConnected()) {
            updateState { it.copy(loadingStatus = LoadingStatus.Connecting) }

            if (connectionBloc.connect(GN2_Address.SYS_DIAG) is Outcome.Error) {
                updateState { it.copy(loadingStatus = LoadingStatus.Error) }
                return@launch
            }
        }

        // ✅ IMPORTANT: block here BEFORE JNI (loadProperties / board collection / poll)
        val blocked = blockIfWrongProduct_Centrella(appContext)
        if (blocked) return@launch

        // Now safe to continue
        deviceBloc.startBoardCollection()
        loadProperties()             // <-- JNI safe now (only on correct product)
        deviceBloc.startBedStatusPoll()
    }
}







private suspend fun blockIfWrongProduct_Dynamo(appContext: Context): Boolean {
    val detected = com.hillrom.servicetool.usb.UsbProductGate.detectDeviceViaUsbPermission(appContext)

    if (detected == null) return false

    if (detected != MedDevices.DYNAMO) {
        updateState {
            it.copy(
                loadingStatus = LoadingStatus.Error,
                incompatibleProductDialogVisible = true,
                incompatibleProductMessage = "Invalid product connected. Please connect the correct device."
            )
        }

        deviceBloc.stopBoardCollection()
        deviceBloc.stopBedStatusPoll()
        connectionBloc.disconnect()

        return true
    }
    return false
}







