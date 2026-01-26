void Firmware::updateBoard(quint32 destination, MEM_MEMORY_TYPE memoryType, QString binPath)
{
    updateComplete[destination]   = false;
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

    ProLog().i(MODULE_NAME, "Firmware upgrade started on " + board.toStdString());

    QString somFileName = "N/A";
    bool toggleBankOnCurrentBoard = false;

    // ---- reset SOM decrypt state every call (minimal, safe) ----
    m_somDecryptPending = false;
    m_somDecryptPath.clear();
    m_somServerUsed = addressForUpdate;

    if (MEM_MEMORY_TYPE_INTERNAL_FLASH == memoryType)
    {
        if (GN2A_SOM == destination)
        {
            // Your code already has binariesFound list.
            // Keep using it. We only fix regex + detect decrypt if present.

            // Correct regex: matches "dynamoSOMApp....tar.gz.enc"
            QRegularExpression tarRx("^dynamoSOMApp.*\\.tar\\.gz\\.enc$");
            int32_t indexOfTarBinary = binariesFound.indexOf(tarRx);

            if (indexOfTarBinary > -1)
            {
                somFileName = binariesFound.at(indexOfTarBinary);

                // Save base folder BEFORE appending filename (so we can build decrypt path)
                QString baseFolder = binPath;

                binPath.append("/");
                binPath.append(somFileName);

                // ---- NEW minimal: detect decrypt file ----
                QRegularExpression decryptRx("^decrypt$");
                int32_t indexOfDecrypt = binariesFound.indexOf(decryptRx);
                if (indexOfDecrypt > -1)
                {
                    m_somDecryptPending = true;
                    m_somDecryptPath = baseFolder + "/" + binariesFound.at(indexOfDecrypt); // "<folder>/decrypt"
                    m_somServerUsed = addressForUpdate; // use same server for decrypt
                }
            }
            else
            {
                // fallback to old behavior
                binPath.append("/");
                binPath.append(internalBinaries[destination]);
            }

            // Do not toggle banks if internal memory.
            toggleBankOnCurrentBoard = false;
        }
        else
        {
            binPath.append("/");
            binPath.append(internalBinaries[destination]);

            // Do not toggle banks if internal memory.
            toggleBankOnCurrentBoard = false;
        }
    }
    else if (MEM_MEMORY_TYPE_EXTERNAL_FLASH0 == memoryType)
    {
        toggleBankOnCurrentBoard = false;
        binPath.append("/");
        binPath.append(externalImages[destination]);
    }
    else
    {
        emit observedError(destination, QString("Unsupported Memory Type"));
        return;
    }

    ProLog().i(MODULE_NAME, "Image path: " + binPath.toStdString());

    FirmwareUpdateCAN* fw = new FirmwareUpdateCAN();
    fw->setImageData(binPath);
    fw->onChangedSelectedServer(addressForUpdate, serverBoard, NULL);
    fw->setDestination(destination);

    // ---- SOM: send filename first (existing behavior) ----
    if (GN2A_SOM == destination)
    {
        MEM_SendImageFileNameRequest  req = MEM_SendImageFileNameRequest_init_zero;
        MEM_SendImageFileNameResponse rsp = MEM_SendImageFileNameResponse_init_zero;

        req.memoryType = memoryType;

        // safer than raw memcpy (ensures null termination)
        QByteArray b = somFileName.toUtf8();
        int n = qMin((int)sizeof(req.imageName) - 1, b.size());
        memcpy(req.imageName, b.constData(), n);
        req.imageName[n] = '\0';

        ProLog().i(MODULE_NAME, "Downloading " + std::string(req.imageName) + " to SOM");

        if (CMD_OK != MEM_SendImageFileName(GN2A_SOM, &req, &rsp))
        {
            ProLog().w(MODULE_NAME, "Could not send file info to " + board.toStdString());
            onBoardUpgraded(fw, destination, false);
            return;
        }
    }
    else
    {
        // existing behavior: validate for non-SOM
        if (!fw->validateImage(memoryType, false)) // do not check board version
        {
            ProLog().w(MODULE_NAME, "Image Verification failed on " + board.toStdString());
            onBoardUpgraded(fw, destination, false);
            return;
        }
    }

    // ---- SIGNALS (keep existing) ----
    // ONLY change: for SOM + decrypt pending, connect to wrapper slot for FIRST file
    if (destination == GN2A_SOM && m_somDecryptPending)
        connect(fw, &FirmwareUpdateCAN::imageUploadCompleted, this, &Firmware::onSomFirstFileDone);
    else
        connect(fw, &FirmwareUpdateCAN::imageUploadCompleted, this, &Firmware::onBoardUpgraded);

    connect(fw, &FirmwareUpdateCAN::errorNotification, this, &Firmware::onObservedError);
    connect(fw, &FirmwareUpdateCAN::infoNotification, this, &Firmware::onObservedInfo);
    connect(fw, &FirmwareUpdateCAN::progressObserved, this, &Firmware::onObservedProgress);
    connect(fw, &FirmwareUpdateCAN::newProgressPhaseStarted, this, &Firmware::onPhaseChanged);
    connect(this, &Firmware::abortUpgrade, fw, &FirmwareUpdateCAN::abortUpgrade, Qt::DirectConnection);

    currentMemoryType = memoryType;
    fw->startUpdate(memoryType, toggleBankOnCurrentBoard);
}
