if (MEM_MEMORY_TYPE_INTERNAL_FLASH == memoryType)
{
    if (GN2A_SOM == destination)
    {
        const QString baseDir = binPath;

        // --- find SOM tar ---
        QRegularExpression tarRx("^dynamoSOMApp.*\\.tar\\.gz\\.enc$");
        int tarIndex = binariesFound.indexOf(tarRx);
        if (tarIndex < 0)
        {
            ProLog().w(MODULE_NAME, "SOM tar not found");
            onBoardUpgraded(nullptr, destination, false);
            return;
        }

        const QString somTarName = binariesFound.at(tarIndex);
        const QString somTarPath = baseDir + "/" + somTarName;

        // --- find decrypt (optional) ---
        QRegularExpression decryptRx("^decrypt(\\..+)?$");
        int decryptIndex = binariesFound.indexOf(decryptRx);

        QString decryptName;
        QString decryptPath;
        if (decryptIndex >= 0)
        {
            decryptName = binariesFound.at(decryptIndex);
            decryptPath = baseDir + "/" + decryptName;
        }

        auto fillReqName = [](MEM_SendImageFileNameRequest &req, const QString &name) {
            memset(req.imageName, 0, sizeof(req.imageName));
            QByteArray n = name.toUtf8();
            const int copyLen = std::min<int>(n.size(), (int)sizeof(req.imageName) - 1);
            memcpy(req.imageName, n.constData(), copyLen);
        };

        auto startTarUpload = [=]() {
            ProLog().i(MODULE_NAME, (QString("Uploading tar: %1").arg(somTarName)).toStdString());

            FirmwareUpdateCAN* fwTar = new FirmwareUpdateCAN();
            // If FirmwareUpdateCAN inherits QObject, this is safe and recommended:
            fwTar->setParent(this);

            fwTar->onChangedSelectedServer(addressForUpdate, serverBoard, NULL);
            fwTar->setDestination(destination);
            fwTar->setImageData(somTarPath);

            MEM_SendImageFileNameRequest req2 = MEM_SendImageFileNameRequest_init_zero;
            MEM_SendImageFileNameResponse rsp2 = MEM_SendImageFileNameResponse_init_zero;
            req2.memoryType = memoryType;
            fillReqName(req2, somTarName);

            if (CMD_OK != MEM_SendImageFileName(GN2A_SOM, &req2, &rsp2))
            {
                ProLog().w(MODULE_NAME, "MEM_SendImageFileName() failed for TAR");
                onBoardUpgraded(fwTar, destination, false);
                return;
            }

            // keep your existing connects for fwTar (error/info/progress) here if you have them:
            // connect(fwTar, &FirmwareUpdateCAN::errorNotification, ...);
            // connect(fwTar, &FirmwareUpdateCAN::infoNotification, ...);
            // connect(fwTar, &FirmwareUpdateCAN::progressObserved, ...);
            // connect(fwTar, &FirmwareUpdateCAN::newProgressPhaseStarted, ...);

            connect(fwTar,
                    &FirmwareUpdateCAN::imageUploadCompleted,
                    this,
                    [=](FirmwareUpdateCAN* /*fw*/, quint32 /*board*/, bool ok2)
                    {
                        ProLog().i(MODULE_NAME, (QString("TAR upload completed ok=%1").arg(ok2)).toStdString());
                        onBoardUpgraded(fwTar, destination, ok2);   // ✅ FINAL completion only here
                    },
                    Qt::QueuedConnection | Qt::SingleShotConnection);

            fwTar->startUpdate(memoryType, false);
        };

        // ---- CASE 1: decrypt exists ----
        if (!decryptName.isEmpty())
        {
            ProLog().i(MODULE_NAME, (QString("Uploading decrypt first: %1").arg(decryptName)).toStdString());

            FirmwareUpdateCAN* fwDecrypt = new FirmwareUpdateCAN();
            fwDecrypt->setParent(this);

            fwDecrypt->onChangedSelectedServer(addressForUpdate, serverBoard, NULL);
            fwDecrypt->setDestination(destination);
            fwDecrypt->setImageData(decryptPath);

            MEM_SendImageFileNameRequest req = MEM_SendImageFileNameRequest_init_zero;
            MEM_SendImageFileNameResponse rsp = MEM_SendImageFileNameResponse_init_zero;
            req.memoryType = memoryType;
            fillReqName(req, decryptName);

            if (CMD_OK != MEM_SendImageFileName(GN2A_SOM, &req, &rsp))
            {
                ProLog().w(MODULE_NAME, "MEM_SendImageFileName() failed for decrypt");
                onBoardUpgraded(fwDecrypt, destination, false);
                return;
            }

            // keep your existing connects for fwDecrypt (error/info/progress) here if you have them:
            // connect(fwDecrypt, &FirmwareUpdateCAN::errorNotification, ...);
            // connect(fwDecrypt, &FirmwareUpdateCAN::infoNotification, ...);
            // connect(fwDecrypt, &FirmwareUpdateCAN::progressObserved, ...);
            // connect(fwDecrypt, &FirmwareUpdateCAN::newProgressPhaseStarted, ...);

            connect(fwDecrypt,
                    &FirmwareUpdateCAN::imageUploadCompleted,
                    this,
                    [=](FirmwareUpdateCAN* /*fw*/, quint32 /*board*/, bool ok1)
                    {
                        if (!ok1) {
                            ProLog().w(MODULE_NAME, "Decrypt upload failed");
                            onBoardUpgraded(fwDecrypt, destination, false); // decrypt failed = overall fail
                            return;
                        }

                        ProLog().i(MODULE_NAME, "Decrypt OK, starting TAR upload");
                        startTarUpload();   // ✅ do NOT call onBoardUpgraded() here
                    },
                    Qt::QueuedConnection | Qt::SingleShotConnection);

            fwDecrypt->startUpdate(memoryType, false);
            return; // SOM special path only
        }

        // ---- CASE 2: NO decrypt (old behavior) ----
        startTarUpload();
        return;
    }

    // everything below remains EXACTLY as before (other boards via MCB)
}
