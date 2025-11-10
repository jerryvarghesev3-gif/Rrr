ll




// PackageExtractor.cpp
#include <QFileInfo>
#include <QDir>
#include <QFile>
#include <QDebug>
#include "JlCompress.h"

// Returns true if at least one file exists in dest after extraction
static bool hasFilesIn(const QString& folder) {
    QDir d(folder);
    return !d.entryList(QDir::Files | QDir::NoDotAndDotDot).isEmpty();
}

bool PackageExtractor::extractPackage(const QString sourceFilePath,
                                      const QString destinationFolderPath)
{
    // ---- 1) Validate paths
    const QFileInfo srcInfo(sourceFilePath);
    if (!srcInfo.exists() || !srcInfo.isFile()) {
        qDebug() << "[FW] BAS not found:" << sourceFilePath;
        return false;
    }

    QDir dest(destinationFolderPath);
    if (!dest.exists()) {
        // make sure the directory exists
        if (!QDir().mkpath(destinationFolderPath)) {
            qDebug() << "[FW] Dest not accessible:" << destinationFolderPath;
            return false;
        }
    }

    // ---- 2) First try: treat .bas as a ZIP and extract everything
    // NOTE: extractDir is the most tolerant API for ZIPs
    qDebug() << "[FW] extractPackage src:" << sourceFilePath
             << " dst:" << destinationFolderPath;

    const QStringList extracted = JlCompress::extractDir(sourceFilePath, destinationFolderPath);

    if (!extracted.isEmpty()) {
        qDebug() << "[FW] extracted files:" << extracted.size();
        return true;
    }

    // If extractDir returned empty but the destination already contains files,
    // consider it a success (some devices write directly during unzip).
    if (hasFilesIn(destinationFolderPath)) {
        qDebug() << "[FW] ZIP extraction empty, but destination already contains files."
                 << "Assuming already extracted.";
        return true;
    }

    // ---- 3) Optional fallback: some legacy .bas may actually be tarballs
    // Only attempt a tar fallback if the header smells like USTAR
    QFile f(sourceFilePath);
    if (f.open(QIODevice::ReadOnly)) {
        // USTAR magic is at offset 257, length 5
        f.seek(257);
        const QByteArray magic = f.read(5);
        f.close();

        if (magic == "ustar") {
            qDebug() << "[FW] ZIP extraction empty, trying TAR fallback...";
            // If you have a TAR/GZ helper, call it here. Example:
            // bool ok = TarGzExtractor::extract(sourceFilePath, destinationFolderPath);
            // if (ok || hasFilesIn(destinationFolderPath)) return true;
        }
    }

    qDebug() << "[FW] JlCompress::extractDir returned empty for" << sourceFilePath;
    return false;
}








// PackageExtractor.cpp
#include <QFileInfo>
#include <QDir>
#include <QFile>
#include <QDebug>
#include "JlCompress.h"

// tiny helper: does dest already contain files?
static bool hasFiles(const QString& folder) {
    QDir d(folder);
    return !d.entryList(QDir::Files | QDir::NoDotAndDotDot).isEmpty();
}

/**
 * Unpacks a .bas (ZIP) into destinationFolderPath.
 * - does NOT touch/expand any inner *.tar.gz.enc; they will be written as-is.
 * - returns true when at least one file is present in destinationFolderPath.
 */
bool PackageExtractor::extractPackage(const QString sourceFilePath,
                                      const QString destinationFolderPath)
{
    qDebug() << "[FW] extractPackage src:" << sourceFilePath
             << " dst:" << destinationFolderPath;

    // 0) quick sanity
    QFileInfo srcInfo(sourceFilePath);
    if (!srcInfo.exists() || !srcInfo.isFile() || srcInfo.size() <= 0) {
        qDebug() << "[FW] BAS not found or empty:" << sourceFilePath;
        return false;
    }

    // 1) ensure destination exists
    QDir destDir(destinationFolderPath);
    if (!destDir.exists()) {
        if (!QDir().mkpath(destinationFolderPath)) {
            qDebug() << "[FW] Dest not accessible:" << destinationFolderPath;
            return false;
        }
    }

    // Optional: clean previous run (keep it if Kotlin already cleans)
    // for (const auto& name : destDir.entryList(QDir::Files | QDir::NoDotAndDotDot))
    //     destDir.remove(name);

    // 2) unzip the BAS as a ZIP *directory* into dest
    //    (this writes inner files such as firmware_imx6_dynamo.tar.gz.enc as-is)
    const QStringList extracted = JlCompress::extractDir(sourceFilePath, destinationFolderPath);

    if (!extracted.isEmpty()) {
        qDebug() << "[FW] extracted files:" << extracted.size();
        return true;
    }

    // 3) Some Android builds write out files despite extractDir returning empty
    if (hasFiles(destinationFolderPath)) {
        qDebug() << "[FW] ZIP extraction empty, but destination already contains files. Assuming success.";
        return true;
    }

    qDebug() << "[FW] JlCompress::extractDir returned empty for" << sourceFilePath;
    return false;
}
