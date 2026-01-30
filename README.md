#include <QThread>

void Firmware::onSomFirstFileDone(FirmwareUpdateCAN* fw, quint32 address, bool success)
{
    // Called only for FIRST SOM file completion when decrypt is pending.

    if (!success)
    {
        onBoardUpgraded(fw, address, false);
        return;
    }

    if (!m_somDecryptPending || m_somDecryptPath.isEmpty())
    {
        onBoardUpgraded(fw, address, true);
        return;
    }

    // Consume decrypt pending (avoid loops)
    m_somDecryptPending = false;

    // We are NOT marking complete yet, so delete first fw ourselves
    if (fw != nullptr)
    {
        delete fw;
        fw = nullptr;
    }

    // Start decrypt upload
    FirmwareUpdateCAN* fw2 = new FirmwareUpdateCAN();
    fw2->setImageData(m_somDecryptPath);
    fw2->onChangedSelectedServer(m_somServerUsed, boardsList[m_somServerUsed], NULL);
    fw2->setDestination(GN2A_SOM);

    // Must set somPackageName on SOM BEFORE sending any write packets
    MEM_SendImageFileNameRequest  req = MEM_SendImageFileNameRequest_init_zero;
    MEM_SendImageFileNameResponse rsp = MEM_SendImageFileNameResponse_init_zero;
    req.memoryType = currentMemoryType;

    QByteArray b("decrypt");
    int n = qMin((int)sizeof(req.imageName) - 1, b.size());
    memcpy(req.imageName, b.constData(), n);
    req.imageName[n] = '\0';

    auto rc = MEM_SendImageFileName(GN2A_SOM, &req, &rsp);
    if (CMD_OK != rc)
    {
        onBoardUpgraded(fw2, GN2A_SOM, false);
        return;
    }

    // Barrier: let SOM update somPackageName before START_PACKET arrives
    QThread::msleep(50);

    // Decrypt completion will use your existing onBoardUpgraded()
    connect(fw2, &FirmwareUpdateCAN::imageUploadCompleted, this, &Firmware::onBoardUpgraded);

    connect(fw2, &FirmwareUpdateCAN::errorNotification, this, &Firmware::onObservedError);
    connect(fw2, &FirmwareUpdateCAN::infoNotification, this, &Firmware::onObservedInfo);
    connect(fw2, &FirmwareUpdateCAN::progressObserved, this, &Firmware::onObservedProgress);
    connect(fw2, &FirmwareUpdateCAN::newProgressPhaseStarted, this, &Firmware::onPhaseChanged);
    connect(this, &Firmware::abortUpgrade, fw2, &FirmwareUpdateCAN::abortUpgrade, Qt::DirectConnection);

    fw2->startUpdate(currentMemoryType, false);
}








#include <QThread>

void Firmware::updateBoard(quint32 destination, MEM_MEMORY_TYPE memoryType, QString binPath)
{
    updateComplete[destination]   = false;
    updateSuccessful[destination] = false;

    QString board = boardsList[destination];
    GN2_Address addressForUpdate = GN2A_MCB;
    QString serverBoard;

    RPCA_Plant* rp = RPCA_Plant::getInstance();
    if ((!rp->isConnectedTo(GN2A_MCB)) && (destination == GN2A_ATLAS))
    {
        addressForUpdate = GN2A_ATLAS;
    }

    serverBoard = boardsList[addressForUpdate];
    ProLog().i(MODULE_NAME, "Firmware upgrade started on " + board.toStdString());

    QString somFileName = "N/A";
    bool toggleBankOnCurrentBoard = false;

    // reset decrypt state
    m_somDecryptPending = false;
    m_somDecryptPath.clear();
    m_somServerUsed = addressForUpdate;

    if (MEM_MEMORY_TYPE_INTERNAL_FLASH == memoryType)
    {
        if (GN2A_SOM == destination)
        {
            QRegularExpression tarRx("^dynamoSOMApp.*\\.tar\\.gz\\.enc$");
            int32_t indexOfTarBinary = binariesFound.indexOf(tarRx);

            if (indexOfTarBinary > -1)
            {
                somFileName = binariesFound.at(indexOfTarBinary);

                QString baseFolder = binPath;   // folder only

                binPath.append("/");
                binPath.append(somFileName);    // full path to SOM app

                // detect decrypt
                QRegularExpression decryptRx("^decrypt$");
                int32_t indexOfDecrypt = binariesFound.indexOf(decryptRx);
                if (indexOfDecrypt > -1)
                {
                    m_somDecryptPending = true;
                    m_somDecryptPath = baseFolder + "/" + binariesFound.at(indexOfDecrypt);
                    m_somServerUsed = addressForUpdate;
                }
            }
            else
            {
                binPath.append("/");
                binPath.append(internalBinaries[destination]);
            }

            toggleBankOnCurrentBoard = false;
        }
        else
        {
            binPath.append("/");
            binPath.append(internalBinaries[destination]);
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

    if (GN2A_SOM == destination)
    {
        MEM_SendImageFileNameRequest  req = MEM_SendImageFileNameRequest_init_zero;
        MEM_SendImageFileNameResponse rsp = MEM_SendImageFileNameResponse_init_zero;

        req.memoryType = memoryType;

        QByteArray b = somFileName.toUtf8();
        int n = qMin((int)sizeof(req.imageName) - 1, b.size());
        memcpy(req.imageName, b.constData(), n);
        req.imageName[n] = '\0';

        auto rc = MEM_SendImageFileName(GN2A_SOM, &req, &rsp);
        if (CMD_OK != rc)
        {
            ProLog().w(MODULE_NAME, "Could not send file info to " + board.toStdString());
            onBoardUpgraded(fw, destination, false);
            return;
        }

        // Barrier: ensure SOM updates somPackageName before first START_PACKET
        QThread::msleep(50);
    }
    else
    {
        if (!fw->validateImage(memoryType, false))
        {
            ProLog().w(MODULE_NAME, "Image Verification failed on " + board.toStdString());
            onBoardUpgraded(fw, destination, false);
            return;
        }
    }

    // connect completion callback
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






