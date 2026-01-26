void Firmware::onSomFirstFileDone(FirmwareUpdateCAN* fw, quint32 address, bool success)
{
    // This slot is used ONLY for the FIRST SOM file when decrypt is present.

    if (!success)
    {
        // First upload failed -> use existing behavior
        onBoardUpgraded(fw, address, false);
        return;
    }

    // If decrypt is not pending (safety) just finish normally
    if (!m_somDecryptPending || m_somDecryptPath.isEmpty())
    {
        onBoardUpgraded(fw, address, true);
        return;
    }

    // Consume pending decrypt (avoid loops)
    m_somDecryptPending = false;

    // IMPORTANT: We are NOT calling onBoardUpgraded for first file (to avoid marking complete too early)
    // So we must delete fw here to avoid memory leak
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

    // Tell SOM the filename = "decrypt" (so it will not overwrite SOM app file)
    MEM_SendImageFileNameRequest  req = MEM_SendImageFileNameRequest_init_zero;
    MEM_SendImageFileNameResponse rsp = MEM_SendImageFileNameResponse_init_zero;
    req.memoryType = currentMemoryType;

    QByteArray b("decrypt");
    int n = qMin((int)sizeof(req.imageName) - 1, b.size());
    memcpy(req.imageName, b.constData(), n);
    req.imageName[n] = '\0';

    if (CMD_OK != MEM_SendImageFileName(GN2A_SOM, &req, &rsp))
    {
        // If decrypt handshake fails -> treat SOM as failed
        onBoardUpgraded(fw2, GN2A_SOM, false);
        return;
    }

    // Now decrypt completion should mark complete using your existing function
    connect(fw2, &FirmwareUpdateCAN::imageUploadCompleted, this, &Firmware::onBoardUpgraded);

    // Keep same connects as your normal flow (if your project uses them)
    connect(fw2, &FirmwareUpdateCAN::errorNotification, this, &Firmware::onObservedError);
    connect(fw2, &FirmwareUpdateCAN::infoNotification, this, &Firmware::onObservedInfo);
    connect(fw2, &FirmwareUpdateCAN::progressObserved, this, &Firmware::onObservedProgress);
    connect(fw2, &FirmwareUpdateCAN::newProgressPhaseStarted, this, &Firmware::onPhaseChanged);
    connect(this, &Firmware::abortUpgrade, fw2, &FirmwareUpdateCAN::abortUpgrade, Qt::DirectConnection);

    fw2->startUpdate(currentMemoryType, false);  // same as SOM behavior: no bank toggle
}











// Save base folder BEFORE appending filename (so we can build decrypt path)
QString baseFolder = binPath;

// ... your existing code does binPath.append("/") and binPath.append(somFileName);

// Check decrypt file exists (optional)
QRegularExpression decryptRx("^decrypt$");
int idxDecrypt = binariesFound.indexOf(decryptRx);

if (idxDecrypt > -1)
{
    m_somDecryptPending = true;
    m_somDecryptPath = baseFolder + "/" + binariesFound.at(idxDecrypt); // "<folder>/decrypt"
}
else
{
    m_somDecryptPending = false;
    m_somDecryptPath.clear();
}

// Save server used (MCB/ATLAS selection) so decrypt uses same route
m_somServerUsed = addressForUpdate;





