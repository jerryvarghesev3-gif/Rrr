kk


// 3) actual extraction (use JlCompress directly for .zip/.bas)
qDebug() << "[FW] extractPackage src:" << sourceFilePath
         << " dst:" << destinationFolderPath;

QStringList extracted = JlCompress::extractDir(sourceFilePath, destinationFolderPath);

if (extracted.isEmpty()) {
    // If nothing was extracted, check whether the destination already contains files
    // (this happens on a second run to the same folder).
    QDir dstDir(destinationFolderPath);
    QStringList existing = dstDir.entryList(QDir::Files | QDir::NoDotAndDotDot);

    if (!existing.isEmpty()) {
        qDebug() << "[FW] ZIP extraction empty, but destination already contains"
                 << existing.size() << "files. Assuming already extracted.";
        binariesFound = existing;            // keep track for later stages
        return true;                         // <- treat as success
    }

    // Optional: only attempt TAR fallback if the source truly looks like a tarball.
    // Many .bas are zip files; avoid pointless tar calls.
    QFile file(sourceFilePath);
    if (file.open(QIODevice::ReadOnly)) {
        QByteArray magic = file.peek(262);   // enough to sniff ustar header
        const bool looksTar =
            sourceFilePath.endsWith(".tar", Qt::CaseInsensitive) ||
            sourceFilePath.endsWith(".tar.gz", Qt::CaseInsensitive) ||
            magic.contains("ustar");
        file.close();

        if (looksTar) {
            qDebug() << "[FW] ZIP extraction empty, trying TAR fallback...";
            // If you actually have a tar fallback, call it here.
            // If not, just fall through to the error below.
        }
    }

    ProLog().e(MODULE_NAME,
               QString("JLCompress::extractDir returned empty for %1")
               .arg(sourceFilePath).toStdString().c_str());
    return false;
}

// Success path
binariesFound = extracted;                   // track for later stages
qDebug() << "[FW] extracted files:" << extracted.size();
return true;


