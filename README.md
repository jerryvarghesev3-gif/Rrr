if (MEM_MEMORY_TYPE_INTERNAL_FLASH == memoryType)
{
    if (GN2A_SOM == destination)
    {
        // IMPORTANT: binPath is a folder. Keep it untouched.
        const QString baseDir = binPath;

        // Find SOM tar in folder
        QRegularExpression tarRx("^dynamoSOMApp.*\\.tar\\.gz\\.enc$");
        int indexOfTarBinary = binariesFound.indexOf(tarRx);

        if (indexOfTarBinary < 0)
        {
            ProLog().w(MODULE_NAME, "SOM tar not found in folder: " + baseDir.toStdString());
            onBoardUpgraded(nullptr, destination, false);
            return;
        }

        const QString somFileName = binariesFound.at(indexOfTarBinary);
        const QString somTarPath  = baseDir + "/" + somFileName;

        // Optional: find decrypt (decrypt or decrypt.<something>)
        QRegularExpression decryptRx("^decrypt(\\..+)?$");
        int indexOfDecrypt = binariesFound.indexOf(decryptRx);

        QString decryptFileName;
        QString decryptPath;
        if (indexOfDecrypt > -1)
        {
            decryptFileName = binariesFound.at(indexOfDecrypt);
            decryptPath = baseDir + "/" + decryptFileName;
        }

        // ---- Upload SOM tar ----
        ProLog().i(MODULE_NAME, ("Downloading " + somFileName + " to SOM").toStdString());

        FirmwareUpdateCAN* fw = new FirmwareUpdateCAN();
        fw->setImageData(somTarPath);
        fw->onChangedSelectedServer(addressForUpdate, serverBoard, NULL);
        fw->setDestination(destination);

        MEM_SendImageFileNameRequest req = MEM_SendImageFileNameRequest_init_zero;
        MEM_SendImageFileNameResponse rsp = MEM_SendImageFileNameResponse_init_zero;
        req.memoryType = memoryType;
        memcpy(req.imageName, somFileName.toStdString().c_str(), somFileName.size());

        if (CMD_OK != MEM_SendImageFileName(GN2A_SOM, &req, &rsp))
        {
            ProLog().w(MODULE_NAME, "Could not send SOM tar file info");
            onBoardUpgraded(fw, destination, false);
            return;
        }

        // Chain decrypt AFTER tar upload completes (only if decrypt exists)
        connect(fw, &FirmwareUpdateCAN::imageUploadCompleted, this, [=]() {
            if (decryptFileName.isEmpty())
            {
                // no decrypt -> done
                return;
            }

            ProLog().i(MODULE_NAME, ("Downloading " + decryptFileName + " to SOM").toStdString());

            FirmwareUpdateCAN* fw2 = new FirmwareUpdateCAN();
            fw2->setImageData(decryptPath);                 //CORRECT PATH: baseDir/decrypt
            fw2->onChangedSelectedServer(addressForUpdate, serverBoard, NULL);
            fw2->setDestination(destination);

            MEM_SendImageFileNameRequest req2 = MEM_SendImageFileNameRequest_init_zero;
            MEM_SendImageFileNameResponse rsp2 = MEM_SendImageFileNameResponse_init_zero;
            req2.memoryType = memoryType;
            memcpy(req2.imageName, decryptFileName.toStdString().c_str(), decryptFileName.size());

            if (CMD_OK != MEM_SendImageFileName(GN2A_SOM, &req2, &rsp2))
            {
                ProLog().w(MODULE_NAME, "Could not send decrypt file info");
                onBoardUpgraded(fw2, destination, false);
                return;
            }

            // start decrypt upload
            fw2->startUpdate(memoryType, false);
        }, Qt::
        connect(fw, &FirmwareUpdateCAN::imageUploadCompleted, this, &Firmware::onBoardUpgraded);
        connect(fw, &FirmwareUpdateCAN::errorNotification, this, &Firmware::onBoardUpgraded);
        connect(fw, &FirmwareUpdateCAN::infoNotification, this, &Firmware::onObservedInfo);
        connect(fw, &FirmwareUpdateCAN::progressObserved, this, &Firmware::onObservedProgress);
        connect(fw, &FirmwareUpdateCAN::newProgressPhaseStarted, this, &Firmware::onPhaseChanged);

        currentMemoryType = memoryType;
        fw->startUpdate(memoryType, false);
        return;
    }

    // ... existing code for other boards ...
}
