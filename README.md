
if (MEM_MEMORY_TYPE_INTERNAL_FLASH == memoryType)
{
    if (GN2A_SOM == destination)
    {
        // -----------------------
        // 1) Find SOM tar.gz.enc (your existing logic)
        // -----------------------
        QRegularExpression tarRx("dynamoSOMApp*\\.tar\\.gz\\.enc");
        int32_t tarIndex = binariesFound.indexOf(tarRx);
        if (tarIndex < 0) {
            ProLog().w(MODULE_NAME, "SOM tar not found");
            onBoardUpgraded(nullptr, destination, false);
            return;
        }

        QString somTarName = binariesFound.at(tarIndex);
        QString somTarPath = binPath + "/" + somTarName;

        // -----------------------
        // 2) Find decrypt (optional)
        // -----------------------
        QRegularExpression decryptRx("^decrypt(\\..+)?$");
        int32_t decryptIndex = binariesFound.indexOf(decryptRx);

        QString decryptName;
        QString decryptPath;
        if (decryptIndex >= 0) {
            decryptName = binariesFound.at(decryptIndex);
            decryptPath = binPath + "/" + decryptName;
        }

        // -----------------------
        // helper: safe copy into req.imageName
        // -----------------------
        auto fillReqName = [](MEM_SendImageFileNameRequest &req, const QString &name) {
            memset(req.imageName, 0, sizeof(req.imageName));
            QByteArray n = name.toUtf8();
            int copyLen = std::min<int>(n.size(), (int)sizeof(req.imageName) - 1);
            memcpy(req.imageName, n.constData(), copyLen);
        };

        // -----------------------
        // helper: attach your EXISTING connects (keep same as old)
        // -----------------------
        auto attachFwSignals = [this](FirmwareUpdateCAN* fw) {
            connect(fw, &FirmwareUpdateCAN::errorNotification, this, &Firmware::onObservedError);
            connect(fw, &FirmwareUpdateCAN::infoNotification,  this, &Firmware::onObservedInfo);
            connect(fw, &FirmwareUpdateCAN::progressObserved,  this, &Firmware::onObservedProgress);
            connect(fw, &FirmwareUpdateCAN::newProgressPhaseStarted, this, &Firmware::onPhaseChanged);
            connect(this, &Firmware::abortUpgrade, fw, &FirmwareUpdateCAN::abortUpgrade, Qt::DirectConnection);
        };

        // -----------------------
        // CASE A: decrypt exists -> upload decrypt first, then tar
        // -----------------------
        if (!decryptName.isEmpty())
        {
            ProLog().i(MODULE_NAME, (QString("Uploading decrypt first: %1").arg(decryptName)).toStdString());

            // ---- decrypt upload ----
            FirmwareUpdateCAN* fwDecrypt = new FirmwareUpdateCAN();
            fwDecrypt->setImageData(decryptPath);                          // keep old order
            fwDecrypt->onChangedSelectedServer(addressForUpdate, serverBoard, NULL);
            fwDecrypt->setDestination(destination);
            attachFwSignals(fwDecrypt);

            MEM_SendImageFileNameRequest req1 = MEM_SendImageFileNameRequest_init_zero;
            MEM_SendImageFileNameResponse rsp1 = MEM_SendImageFileNameResponse_init_zero;
            req1.memoryType = memoryType;
            fillReqName(req1, decryptName);

            if (CMD_OK != MEM_SendImageFileName(GN2A_SOM, &req1, &rsp1))
            {
                onBoardUpgraded(fwDecrypt, destination, false);
                return;
            }

            // when decrypt finishes -> start TAR upload (DO NOT call onBoardUpgraded here)
            connect(
                fwDecrypt,
                &FirmwareUpdateCAN::imageUploadCompleted,
                this,
                [=](FirmwareUpdateCAN* /*fw*/, quint32 /*board*/, bool ok1)
                {
                    if (!ok1) {
                        ProLog().w(MODULE_NAME, "Decrypt upload failed");
                        onBoardUpgraded(fwDecrypt, destination, false);
                        return;
                    }

                    ProLog().i(MODULE_NAME, (QString("Decrypt OK. Uploading tar second: %1").arg(somTarName)).toStdString());

                    // ---- TAR upload ----
                    FirmwareUpdateCAN* fwTar = new FirmwareUpdateCAN();
                    fwTar->setImageData(somTarPath);                       // keep old order
                    fwTar->onChangedSelectedServer(addressForUpdate, serverBoard, NULL);
                    fwTar->setDestination(destination);
                    attachFwSignals(fwTar);

                    MEM_SendImageFileNameRequest req2 = MEM_SendImageFileNameRequest_init_zero;
                    MEM_SendImageFileNameResponse rsp2 = MEM_SendImageFileNameResponse_init_zero;
                    req2.memoryType = memoryType;
                    fillReqName(req2, somTarName);

                    if (CMD_OK != MEM_SendImageFileName(GN2A_SOM, &req2, &rsp2))
                    {
                        onBoardUpgraded(fwTar, destination, false);
                        return;
                    }

                    // FINAL completion only when TAR upload completes
                    connect(
                        fwTar,
                        &FirmwareUpdateCAN::imageUploadCompleted,
                        this,
                        [=](FirmwareUpdateCAN* /*fw*/, quint32 /*board*/, bool ok2)
                        {
                            ProLog().i(MODULE_NAME, (QString("TAR upload completed ok=%1").arg(ok2)).toStdString());
                            onBoardUpgraded(fwTar, destination, ok2);
                        },
                        Qt::QueuedConnection
                    );

                    currentMemoryType = memoryType;
                    fwTar->startUpdate(memoryType, false);
                },
                Qt::QueuedConnection
            );

            currentMemoryType = memoryType;
            fwDecrypt->startUpdate(memoryType, false);
            return; // SOM handled here only
        }

        // -----------------------
        // CASE B: no decrypt -> OLD behaviour (tar only)
        // -----------------------
        ProLog().i(MODULE_NAME, (QString("Uploading tar only: %1").arg(somTarName)).toStdString());

        FirmwareUpdateCAN* fw = new FirmwareUpdateCAN();
        fw->setImageData(somTarPath);                                     // keep old order
        fw->onChangedSelectedServer(addressForUpdate, serverBoard, NULL);
        fw->setDestination(destination);
        attachFwSignals(fw);

        MEM_SendImageFileNameRequest req = MEM_SendImageFileNameRequest_init_zero;
        MEM_SendImageFileNameResponse rsp = MEM_SendImageFileNameResponse_init_zero;
        req.memoryType = memoryType;
        fillReqName(req, somTarName);

        if (CMD_OK != MEM_SendImageFileName(GN2A_SOM, &req, &rsp))
        {
            onBoardUpgraded(fw, destination, false);
            return;
        }

        currentMemoryType = memoryType;
        fw->startUpdate(memoryType, false);
        return; // SOM handled here only
    }

    // ✅ below this line: keep your old code EXACTLY (other boards via MCB)
}
















if (GN2A_SOM == destination)
{
    QRegularExpression tarRx = QRegularExpression("dynamoSOMApp*\\.tar.gz.enc");
    int32_t indexOfTarBinary = binariesFound.indexOf(tarRx);

    if (indexOfTarBinary > -1)
    {
        // build TAR path (old logic)
        somFileName = binariesFound.at(indexOfTarBinary);

        QString baseDir = binPath;     // keep base dir before appending file
        baseDir.append("/");

        QString tarPath = baseDir + somFileName;

        // --- NEW: find decrypt optional ---
        QRegularExpression decryptRx("^decrypt(\\..+)?$");
        int32_t decryptIndex = binariesFound.indexOf(decryptRx);

        if (decryptIndex > -1)
        {
            QString decryptName = binariesFound.at(decryptIndex);
            QString decryptPath = baseDir + decryptName;

            // store TAR for later (after decrypt completes)
            somPendingTar     = true;
            somPendingTarName = somFileName;
            somPendingTarPath = tarPath;

            // swap: first upload decrypt
            somFileName = decryptName;
            binPath = decryptPath;

            ProLog().i(MODULE_NAME, ("Uploading decrypt first: " + somFileName).toStdString());
        }
        else
        {
            // no decrypt -> normal TAR flow
            binPath = tarPath;
        }
    }
}
