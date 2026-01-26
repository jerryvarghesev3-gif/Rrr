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

        // helper: safe imageName copy (null-terminated)
        auto fillReqName = [](MEM_SendImageFileNameRequest &req, const QString &name) {
            memset(req.imageName, 0, sizeof(req.imageName));
            QByteArray n = name.toUtf8();
            const int copyLen = std::min<int>(n.size(), (int)sizeof(req.imageName) - 1);
            memcpy(req.imageName, n.constData(), copyLen);
        };

        // -------- CASE 1: decrypt exists -> upload decrypt first, then tar --------
        if (!decryptName.isEmpty())
        {
            ProLog().i(MODULE_NAME, (QString("Uploading decrypt first: %1").arg(decryptName)).toStdString());

            // IMPORTANT: parent to 'this' so lifetime is safe
            QPointer<FirmwareUpdateCAN> fwDecrypt = new FirmwareUpdateCAN(this);
            fwDecrypt->onChangedSelectedServer(addressForUpdate, serverBoard, NULL);
            fwDecrypt->setDestination(destination);
            fwDecrypt->setImageData(decryptPath);

            MEM_SendImageFileNameRequest req = MEM_SendImageFileNameRequest_init_zero;
            MEM_SendImageFileNameResponse rsp = MEM_SendImageFileNameResponse_init_zero;
            req.memoryType = memoryType;
            fillReqName(req, decryptName);

            if (CMD_OK != MEM_SendImageFileName(GN2A_SOM, &req, &rsp))
            {
                ProLog().w(MODULE_NAME, "MEM_SendImageFileName failed for decrypt");
                onBoardUpgraded(fwDecrypt, destination, false);
                return;
            }

            connect(fwDecrypt,
                    &FirmwareUpdateCAN::imageUploadCompleted,
                    this,
                    [=](FirmwareUpdateCAN* /*fw*/, quint32 /*board*/, bool ok1)
                    {
                        if (!fwDecrypt) return; // was deleted somewhere
                        if (!ok1)
                        {
                            ProLog().w(MODULE_NAME, "Decrypt upload failed");
                            onBoardUpgraded(fwDecrypt, destination, false);
                            fwDecrypt->deleteLater();
                            return;
                        }

                        ProLog().i(MODULE_NAME, (QString("Decrypt OK. Uploading tar second: %1").arg(somTarName)).toStdString());

                        // TAR upload - separate object, also parented
                        QPointer<FirmwareUpdateCAN> fwTar = new FirmwareUpdateCAN(this);
                        fwTar->onChangedSelectedServer(addressForUpdate, serverBoard, NULL);
                        fwTar->setDestination(destination);
                        fwTar->setImageData(somTarPath);

                        MEM_SendImageFileNameRequest req2 = MEM_SendImageFileNameRequest_init_zero;
                        MEM_SendImageFileNameResponse rsp2 = MEM_SendImageFileNameResponse_init_zero;
                        req2.memoryType = memoryType;
                        fillReqName(req2, somTarName);

                        if (CMD_OK != MEM_SendImageFileName(GN2A_SOM, &req2, &rsp2))
                        {
                            ProLog().w(MODULE_NAME, "MEM_SendImageFileName failed for tar");
                            onBoardUpgraded(fwTar, destination, false);
                            if (fwTar) fwTar->deleteLater();
                            fwDecrypt->deleteLater();
                            return;
                        }

                        connect(fwTar,
                                &FirmwareUpdateCAN::imageUploadCompleted,
                                this,
                                [=](FirmwareUpdateCAN* /*fw*/, quint32 /*board*/, bool ok2)
                                {
                                    if (!fwTar) return;
                                    ProLog().i(MODULE_NAME, (QString("TAR upload completed ok=%1").arg(ok2)).toStdString());
                                    onBoardUpgraded(fwTar, destination, ok2);
                                    fwTar->deleteLater();
                                },
                                Qt::QueuedConnection);

                        // start TAR transfer
                        fwTar->startUpdate(memoryType, false);

                        // cleanup decrypt object after tar started
                        fwDecrypt->deleteLater();
                    },
                    Qt::QueuedConnection);

            // start decrypt transfer
            fwDecrypt->startUpdate(memoryType, false);
            return; // SOM special path only
        }

        // -------- CASE 2: no decrypt -> old behavior (tar only) --------
        ProLog().i(MODULE_NAME, (QString("Uploading tar only: %1").arg(somTarName)).toStdString());

        QPointer<FirmwareUpdateCAN> fw = new FirmwareUpdateCAN(this);
        fw->onChangedSelectedServer(addressForUpdate, serverBoard, NULL);
        fw->setDestination(destination);
        fw->setImageData(somTarPath);

        MEM_SendImageFileNameRequest req = MEM_SendImageFileNameRequest_init_zero;
        MEM_SendImageFileNameResponse rsp = MEM_SendImageFileNameResponse_init_zero;
        req.memoryType = memoryType;
        fillReqName(req, somTarName);

        if (CMD_OK != MEM_SendImageFileName(GN2A_SOM, &req, &rsp))
        {
            ProLog().w(MODULE_NAME, "MEM_SendImageFileName failed for tar-only case");
            onBoardUpgraded(fw, destination, false);
            if (fw) fw->deleteLater();
            return;
        }

        connect(fw,
                &FirmwareUpdateCAN::imageUploadCompleted,
                this,
                [=](FirmwareUpdateCAN* /*fw*/, quint32 /*board*/, bool ok)
                {
                    if (!fw) return;
                    ProLog().i(MODULE_NAME, (QString("TAR upload completed ok=%1").arg(ok)).toStdString());
                    onBoardUpgraded(fw, destination, ok);
                    fw->deleteLater();
                },
                Qt::QueuedConnection);

        fw->startUpdate(memoryType, false);
        return;
    }

    // ✅ everything below remains EXACTLY as before (other boards via MCB)
}
