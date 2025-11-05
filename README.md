hh



QStringList extracted;

// Try as ZIP first
extracted = JlCompress::extractDir(sourceFilePath, destinationFolderPath);
if (extracted.isEmpty()) {
    qDebug() << "[FW] ZIP extraction empty, trying TAR fallback...";
    QProcess::execute("/system/bin/sh", {"-c",
        QString("tar -xf %1 -C %2").arg(sourceFilePath).arg(destinationFolderPath)});
    QDir dir(destinationFolderPath);
    extracted = dir.entryList(QDir::Files | QDir::NoDotAndDotDot);
}

if (extracted.isEmpty()) {
    ProLog().e(MODULE_NAME,
               qPrintable(QString("Extraction failed for %1").arg(sourceFilePath)));
    return false;
}
