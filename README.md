Dear Hiring Team,

I am a Senior Embedded / C++ Software Engineer with 7.5+ years of experience working on performance-critical systems using modern C++ (C++14/17), Embedded Linux, Yocto, BSP and low-level system integration.

I have worked across the full product lifecycle, from low-level firmware and device interaction up to system integration, testing and deployment, particularly in complex embedded and IoT environments.

I am highly motivated to relocate to Switzerland and am open to employer-sponsored work permits. I am currently learning German and plan to reach B2 level, while being fully fluent in English for professional communication.

I believe my system-level mindset, strong C++ background and experience with real-world embedded products align well with PXL Vision’s work in high-quality imaging and vision systems.

Thank you for your time and consideration.

Kind regards,
Jerry Varghese











I am currently based in India and fully willing to relocate to Zurich.
I am available to start from April 2026 and am open to supporting the visa
and relocation process.

I bring strong experience in embedded C++ and Linux development, working close
to hardware and collaborating with FPGA and hardware teams.
I am particularly motivated by Zurich Instruments’ advanced R&D work
in high-performance measurement systems.

I am actively learning German and plan to complete B2 certification in 2026.







private fun load() = launch {

    // Step 1: connect (safe)
    if (!connectionBloc.isConnected()) {
        updateState { it.copy(loadingStatus = LoadingStatus.Connecting) }

        if (connectionBloc.connect(GN2_Address.SYS_DIAG) is Outcome.Error) {
            updateState { it.copy(loadingStatus = LoadingStatus.Error) }
            return@launch
        }
    }

    // Step 2: SAFE product check (NO Dynamo calls yet)
    val detectedDevice = detectDeviceFromModel(state.modelNumber)

    if (detectedDevice != MedDevices.DYNAMO) {
        updateState {
            it.copy(
                loadingStatus = LoadingStatus.Error,
                incompatibleProductDialogVisible = true,
                incompatibleProductMessage =
                    "Invalid product connected. Please connect the correct device."
            )
        }

        // Important: STOP HERE
        connectionBloc.disconnect()
        return@launch
    }

    // Step 3: ONLY NOW it is safe to touch Dynamo native code
    deviceBloc.startBoardCollection()
    loadProperties()                 // can call Dynamo JNI now
    deviceBloc.startBedStatusPoll()
}








// connection already successful here

val detected = detectFromUsbOrHandshake() // 🚫 NO JNI here

if (detected != MedDevices.DYNAMO) {
    updateState {
        it.copy(
            loadingStatus = LoadingStatus.Error,
            incompatibleProductDialogVisible = true,
            incompatibleProductMessage =
                "Invalid product connected. Please connect the correct device."
        )
    }

    connectionBloc.disconnect()
    return@launch
}





private fun detectFromUsbOrHandshake(): MedDevices? {
    // Option 1: USB VID / PID
    detectDeviceFromUsb(appContext)?.let { return it }

    // Option 2: cached handshake info (string-based only)
    connectionBloc.lastKnownProduct?.let { return it }

    // Option 3: unknown → allow flow to continue
    return null
}





