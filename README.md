hh
#include <QFileInfo>
#include <QDir>
#include <JlCompress.h>

static QString lastJlError; // optional, if you want to retain a message

bool Firmware::extractPackage(QString sourceFilePath, QString destinationFolderPath)
{
    qDebug() << "[FW] extractPackage src:" << sourceFilePath
             << " dst:" << destinationFolderPath;

    // 1) sanity checks
    QFileInfo srcInfo(sourceFilePath);
    if (!srcInfo.exists() || !srcInfo.isFile()) {
        ProLog().e(MODULE_NAME, QString("BAS not found: %1").arg(sourceFilePath));
        return false;
    }

    // 2) ensure destination is writable
    QDir().mkpath(destinationFolderPath);
    QDir dst(destinationFolderPath);
    if (!dst.exists()) {
        ProLog().e(MODULE_NAME, QString("Dest not accessible: %1").arg(destinationFolderPath));
        return false;
    }

    // 3) actual extraction (use JLCompress directly)
    //    It handles .zip (your .bas). It returns list of extracted files.
    QStringList extracted = JLCompress::extractDir(sourceFilePath, destinationFolderPath);

    if (extracted.isEmpty()) {
        ProLog().e(MODULE_NAME,
                   QString("JLCompress::extractDir returned empty for %1").arg(sourceFilePath));
        return false;
    }

    // (Optional) keep track of what we found for next stages
    binariesFound = extracted;

    qDebug() << "[FW] extracted files:" << extracted.size();
    return true;
}
