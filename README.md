ll




// helper
static bool hasFilesIn(const QString& folder) {
    QDir d(folder);
    return !d.entryList(QDir::Files | QDir::NoDotAndDotDot).isEmpty();
}

// tolerant extractor
bool PackageExtractor::extractPackage(const QString sourceFilePath,
                                      const QString destinationFolderPath)
{
    // 1) validate paths
    QFileInfo srcInfo(sourceFilePath);
    if (!srcInfo.exists() || !srcInfo.isFile()) {
        qDebug() << "[FW] BAS not found:" << sourceFilePath;
        return false;
    }
    QDir dest(destinationFolderPath);
    if (!dest.exists()) {
        if (!QDir().mkpath(destinationFolderPath)) {
            qDebug() << "[FW] Dest not accessible:" << destinationFolderPath;
            return false;
        }
    }

    // 2) try ZIP first (JlCompress is tolerant for .zip/.bas)
    qDebug() << "[FW] extractPackage src:" << sourceFilePath
             << " dst:" << destinationFolderPath;
    const QStringList extracted = JlCompress::extractDir(sourceFilePath, destinationFolderPath);
    if (!extracted.isEmpty()) {
        qDebug() << "[FW] extracted files:" << extracted.size();
        return true;
    }

    // 2b) if API reports empty BUT destination already has files, assume success
    if (hasFilesIn(destinationFolderPath)) {
        qDebug() << "[FW] ZIP extraction empty, but destination already contains files. Assuming already extracted.";
        return true;
    }

    // 3) optional TAR fallback (legacy .bas that are tarballs)
    QFile f(sourceFilePath);
    if (f.open(QIODevice::ReadOnly)) {
        // 'ustar' magic at offset 257 (len 5)
        f.seek(257);
        const QByteArray magic = f.read(5);
        f.close();
        if (magic == "ustar") {
            qDebug() << "[FW] ZIP extraction empty, trying TAR fallback...";
            // If you have a tar/gz helper, call it here and return true on success.
            // bool ok = TarGzExtractor::extract(sourceFilePath, destinationFolderPath);
            // if (ok || hasFilesIn(destinationFolderPath)) return true;
        }
    }

    qDebug() << "[FW] JlCompress::extractDir returned empty for" << sourceFilePath;
    return false;
}
