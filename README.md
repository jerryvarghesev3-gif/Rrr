if (GN2A_SOM == destination)
{
    QRegularExpression tarRx("dynamoSOMApp.*\\.tar\\.gz\\.enc");   // IMPORTANT: correct regex
    int32_t indexOfTarBinary = binariesFound.indexOf(tarRx);

    if (indexOfTarBinary > -1)
    {
        somFileName = binariesFound.at(indexOfTarBinary);

        QString baseDir = binPath;
        baseDir.append("/");

        QString tarPath = baseDir + somFileName;

        // --- NEW: find decrypt optional ---
        QRegularExpression decryptRx("^decrypt(\\..+)?$");
        int32_t decryptIndex = binariesFound.indexOf(decryptRx);

        if (decryptIndex > -1)
        {
            QString decryptName = binariesFound.at(decryptIndex);
            QString decryptPath = baseDir + decryptName;

            // ✅ store TAR details for after decrypt completes
            somPendingTar = true;
            somPendingTarName = somFileName;
            somPendingTarPath = tarPath;
            somPendingMemoryType = memoryType;

            // ✅ swap to upload decrypt first
            somFileName = decryptName;
            binPath = decryptPath;

            ProLog().i(MODULE_NAME, ("Uploading decrypt first: " + somFileName).toStdString());
        }
        else
        {
            // normal TAR flow
            binPath = tarPath;
            somPendingTar = false;
        }
    }
}
