enum class DynamoYoctoOS(val yoctoPrjVer: DSemVer) {
    INVALID(DSemVer(0, 0, 0, 0)),

    //Dunfell is 1.0.0.0 (this is correct base)
    DUNFELL(DSemVer(1, 0, 0, 0));

    companion object {
        fun fromVersion(findVer: DSemVer): DynamoYoctoOS {
            // treat all-zero as invalid
            if (findVer.major == 0 && findVer.minor == 0 && findVer.patch == 0 && findVer.build == 0) {
                return INVALID
            }

            //match ALL 4 parts (important!)
            return values().firstOrNull { yocto ->
                yocto.yoctoPrjVer.major == findVer.major &&
                yocto.yoctoPrjVer.minor == findVer.minor &&
                yocto.yoctoPrjVer.patch == findVer.patch &&
                yocto.yoctoPrjVer.build == findVer.build
            } ?: INVALID
        }
    }
}






val somFwSemVer = state.boardStatuses[DynamoBoard.SYSTEM_ON_MODULE]?.firmwareVersion

val somFwDVer = somFwSemVer
    ?.let { DSemVer.fromString(it.toString()) }
    ?: DSemVer()   // 0.0.0.0 fallback

fwBloc.evaluateFirmwareCompatibility(
    firmwareFileName = selectedFile.name ?: "",
    firmwareFileInputStream = inputStream,
    somVersion = somFwDVer,
    isBasicMode = isBasic,
    boardsToUpdate = if (isBasic) null else state.advancedBoardList
)









override suspend fun evaluateFirmwareCompatibility(
    activeOsVersion: DynamoYoctoOS, // keep it only for logging/validation
    releaseManifest: SoftwareReleaseManifest,
): Outcome<DynamoFirmwareCompatibilityStatus> {

    val files = extractionFolder.listFiles().orEmpty()

    // OS file can be tar.gz OR tar.gz.enc
    val containsOsTar = files.any {
        it.name.equals("firmware_imx6_dynamo.tar.gz", ignoreCase = true) ||
        it.name.equals("firmware_imx6_dynamo.tar.gz.enc", ignoreCase = true)
    }

    ProLog.i(
        MODULE_NAME,
        "Evaluating FW compatibility: detectedYocto=$activeOsVersion, extractedFiles=${files.map { it.name }}"
    )
    ProLog.i(MODULE_NAME, "OS package presence check: containsOsTar=$containsOsTar")

    if (!containsOsTar) {
        ProLog.i(MODULE_NAME, "OS not included -> OSUpdateNotIncluded")
        return Outcome.Ok(DynamoFirmwareCompatibilityStatus.OSUpdateNotIncluded)
    }

    // ✅ single source of truth: manifest vs device OS diff
    val manifestDecision = shouldSendOsFromManifest(releaseManifest)

    ProLog.i(MODULE_NAME, "Manifest decision (manifest OS != device OS): $manifestDecision")

    return if (manifestDecision) {
        ProLog.i(MODULE_NAME, "OSUpdateIncluded -> OS WILL be sent")
        Outcome.Ok(DynamoFirmwareCompatibilityStatus.OSUpdateIncluded)
    } else {
        ProLog.i(MODULE_NAME, "OSUpdateNotIncluded -> OS will NOT be sent")
        Outcome.Ok(DynamoFirmwareCompatibilityStatus.OSUpdateNotIncluded)
    }
}





override suspend fun evaluateFirmwareCompatibility(
    activeOsVersion: DynamoYoctoOS,
    releaseManifest: SoftwareReleaseManifest,
): Outcome<DynamoFirmwareCompatibilityStatus> {

    val files = extractionFolder.listFiles().orEmpty()
    val fileNames = files.map { it.name }

    ProLog.i(
        MODULE_NAME,
        msg = "Evaluating fw compatibility: activeOsVersion=$activeOsVersion, extractedFiles=$fileNames"
    )

    // OS file can be either tar.gz OR tar.gz.enc
    val containsOsTar = files.any {
        it.name.equals("firmware_imx6_dynamo.tar.gz", ignoreCase = true) ||
        it.name.equals("firmware_imx6_dynamo.tar.gz.enc", ignoreCase = true)
    }

    ProLog.i(MODULE_NAME, msg = "OS presence: containsOsTar=$containsOsTar")

    if (!containsOsTar) {
        ProLog.i(MODULE_NAME, msg = "OS not included -> OSUpdateNotIncluded")
        return Outcome.Ok(DynamoFirmwareCompatibilityStatus.OSUpdateNotIncluded)
    }

    val manifestDecision = shouldSendOsFromManifest(releaseManifest)
    ProLog.i(MODULE_NAME, msg = "Manifest decision (shouldSendOsFromManifest)=$manifestDecision")

    val activeIsDunfell = (activeOsVersion == DynamoYoctoOS.DUNFELL)
    ProLog.i(MODULE_NAME, msg = "activeOsIsDunfell=$activeIsDunfell")

    val shouldSendOsUpdate = activeIsDunfell && manifestDecision
    ProLog.i(MODULE_NAME, msg = "Final shouldSendOsUpdate=$shouldSendOsUpdate")

    return if (shouldSendOsUpdate) {
        ProLog.i(MODULE_NAME, msg = "OSUpdateIncluded -> OS WILL be sent")
        Outcome.Ok(DynamoFirmwareCompatibilityStatus.OSUpdateIncluded)
    } else {
        ProLog.i(MODULE_NAME, msg = "OSUpdateNotIncluded -> OS will NOT be sent")
        Outcome.Ok(DynamoFirmwareCompatibilityStatus.OSUpdateNotIncluded)
    }
}

















void Firmware::updateBoard(quint32 destination, MEM_MEMORY_TYPE memoryType, QString binPath)
{
    updateComplete[destination] = false;
    updateSuccessful[destination] = false;

    QString board = boardsList[destination];
    GN2_Address addressForUpdate = GN2A_MCB;
    QString serverBoard;

    // In case MCB is not connected and we are upgrading ATLAS, it could be stand alone atlas
    RPCA_Plant* rp = RPCA_Plant::getInstance();
    if ((!rp->isConnectedTo(GN2A_MCB)) && (destination == GN2A_ATLAS))
    {
        addressForUpdate = GN2A_ATLAS;
    }

    serverBoard = boardsList[addressForUpdate];

    ProLog().i(MODULE_NAME, ("Firmware upgrade started on " + board).toStdString());

    QString somFileName = "N/A";

    // -------------------------------
    // Build full image path used by CAN uploader
    // -------------------------------
    if (MEM_MEMORY_TYPE_INTERNAL_FLASH == memoryType)
    {
        // SOM special handling OR normal internalBinaries handling
        if (GN2A_SOM == destination)
        {
            // ✅ Find SOM archive
            QRegularExpression tarRx("dynamoSOMApp.*\\.tar\\.gz\\.enc", QRegularExpression::CaseInsensitiveOption);
            int indexOfTarBinary = binariesFound.indexOf(tarRx);

            if (indexOfTarBinary > -1)
            {
                somFileName = binariesFound.at(indexOfTarBinary);
            }
            else
            {
                ProLog().w(MODULE_NAME, "Could not find dynamoSOMApp*.tar.gz.enc in binariesFound");
                onBoardUpgraded(nullptr, destination, false);
                return;
            }

            // ✅ binPath should point to folder; build full path to SOM archive
            binPath.append("/");
            binPath.append(somFileName);

            ProLog().i(MODULE_NAME, ("Image path: " + binPath).toStdString());
        }
        else
        {
            // Normal internal binaries for non-SOM
            binPath.append("/");
            binPath.append(internalBinaries[destination]);
        }

        // Do not toggle the banks if internal memory.
        toggleBankOnCurrentBoard = false;
    }
    else if (MEM_MEMORY_TYPE_EXTERNAL_FLASH0 == memoryType)
    {
        toggleBankOnCurrentBoard = false;
        binPath.append("/");
        binPath.append(externalImages[destination]);
    }

    ProLog().i(MODULE_NAME, ("Image path: " + binPath).toStdString());

    FirmwareUpdateCAN* fw = new FirmwareUpdateCAN();
    fw->setImageData(binPath);
    fw->onChangedSelectedServer(addressForUpdate, serverBoard, nullptr);
    fw->setDestination(destination);

    // ----------------------------------------
    // ✅ SOM special: announce filenames to SOM
    // ----------------------------------------
    if (GN2A_SOM == destination)
    {
        MEM_SendImageFileNameRequest req = MEM_SendImageFileNameRequest_init_zero;
        MEM_SendImageFileNameResponse rsp = MEM_SendImageFileNameResponse_init_zero;

        req.memoryType = memoryType;

        // ✅ Send SOM tar.gz.enc filename first
        memset(req.imageName, 0, sizeof(req.imageName));
        QByteArray somBytes = somFileName.toUtf8();
        strncpy(req.imageName, somBytes.constData(), sizeof(req.imageName) - 1);

        ProLog().i(MODULE_NAME, ("Downloading " + somFileName + " to SOM").toStdString());

        if (CMD_OK != MEM_SendImageFileName(GN2A_SOM, &req, &rsp))
        {
            ProLog().w(MODULE_NAME, ("Could not send file info to " + board).toStdString());
            onBoardUpgraded(fw, destination, false);
            return;
        }

        // ✅ Now detect decrypt (decrypt or decrypt.*) and send it too
        // This matches your Android logic: equals "decrypt" OR startsWith("decrypt.")
        QRegularExpression decryptRx("^decrypt(\\..*)?$", QRegularExpression::CaseInsensitiveOption);
        int indexOfDecrypt = binariesFound.indexOf(decryptRx);

        if (indexOfDecrypt > -1)
        {
            QString decryptName = binariesFound.at(indexOfDecrypt);

            MEM_SendImageFileNameRequest req2 = MEM_SendImageFileNameRequest_init_zero;
            MEM_SendImageFileNameResponse rsp2 = MEM_SendImageFileNameResponse_init_zero;

            req2.memoryType = memoryType;

            memset(req2.imageName, 0, sizeof(req2.imageName));
            QByteArray decBytes = decryptName.toUtf8();
            strncpy(req2.imageName, decBytes.constData(), sizeof(req2.imageName) - 1);

            ProLog().i(MODULE_NAME, ("Downloading " + decryptName + " to SOM").toStdString());

            if (CMD_OK != MEM_SendImageFileName(GN2A_SOM, &req2, &rsp2))
            {
                // Don’t hard-fail if decrypt fails (optional), but better to fail if your SOM requires it.
                ProLog().w(MODULE_NAME, "Failed to send decrypt filename to SOM");
                onBoardUpgraded(fw, destination, false);
                return;
            }
        }
        else
        {
            ProLog().w(MODULE_NAME, "decrypt not found in binariesFound (expected decrypt or decrypt.*)");
            // If decrypt is mandatory for SOM packages, you can fail here instead:
            // onBoardUpgraded(fw, destination, false);
            // return;
        }
    }

    // ----------------------------------------
    // Validate and start update (existing logic)
    // ----------------------------------------
    if (!fw->validateImage(memoryType, false)) // do not check board version
    {
        ProLog().w(MODULE_NAME, ("Image Verification failed on " + board).toStdString());
        onBoardUpgraded(fw, destination, false);
        return;
    }

    connect(fw, &FirmwareUpdateCAN::imageUploadCompleted, this, &Firmware::onBoardUpgraded);
    connect(fw, &FirmwareUpdateCAN::errorNotification, this, &Firmware::onObservedError);
    connect(fw, &FirmwareUpdateCAN::infoNotification, this, &Firmware::onObservedInfo);
    connect(fw, &FirmwareUpdateCAN::progressObserved, this, &Firmware::onObservedProgress);
    connect(fw, &FirmwareUpdateCAN::newProgressPhaseStarted, this, &Firmware::onPhaseChanged);
    connect(this, &Firmware::abortUpgrade, fw, &FirmwareUpdateCAN::abortUpgrade, Qt::DirectConnection);

    currentMemoryType = memoryType;
    fw->startUpdate(memoryType, toggleBankOnCurrentBoard);
}
