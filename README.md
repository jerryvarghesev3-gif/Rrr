if (GN2A_SOM == destination)
{
    // 1) find TAR
    QRegularExpression tarRx("dynamoSOMApp.*\\.tar\\.gz\\.enc");
    int32_t tarIndex = binariesFound.indexOf(tarRx);
    if (tarIndex < 0) {
        ProLog().w(MODULE_NAME, "SOM tar not found");
        onBoardUpgraded(nullptr, destination, false);
        return;
    }

    QString tarName = binariesFound.at(tarIndex);

    QString baseDir = binPath;
    baseDir.append("/");

    QString tarPath = baseDir + tarName;

    // 2) find decrypt (optional)
    QRegularExpression decryptRx("^decrypt(\\..+)?$");
    int32_t decryptIndex = binariesFound.indexOf(decryptRx);

    if (decryptIndex > -1)
    {
        QString decryptName = binariesFound.at(decryptIndex);
        QString decryptPath = baseDir + decryptName;

        // store TAR for later
        somPendingTar = true;
        somPendingTarName = tarName;
        somPendingTarPath = tarPath;
        somPendingMemoryType = memoryType;

        // swap to decrypt first
        somFileName = decryptName;
        binPath = decryptPath;

        ProLog().i(MODULE_NAME, ("Uploading decrypt first: " + somFileName).toStdString());
    }
    else
    {
        // normal TAR flow
        somPendingTar = false;
        somFileName = tarName;
        binPath = tarPath;
    }
}
