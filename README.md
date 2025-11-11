ll





// PackageExtractor.h
static bool hasFilesIn(const QString& folder);

// PackageExtractor.cpp
#include <QFileInfo>
#include <QDir>
#include <QFile>
#include <QByteArray>

static bool hasFilesIn(const QString& folder) {
    QDir d(folder);
    return !d.entryList(QDir::Files | QDir::NoDotAndDotDot).isEmpty();
}

bool PackageExtractor::extractPackage(const QString sourceFilePath,
                                      const QString destinationFolderPath)
{
    // -- 1) Validate inputs
    QFileInfo srcInfo(sourceFilePath);
    if (!srcInfo.exists() || !srcInfo.isFile()) {
        qDebug() << "[FW] BAS not found or empty:" << sourceFilePath;
        return false;
    }

    QDir dest(destinationFolderPath);
    if (!dest.exists()) {
        if (!dest.mkpath(destinationFolderPath)) {
            qDebug() << "[FW] Dest not accessible:" << destinationFolderPath;
            return false;
        }
    }

    // -- 2) First try: treat .bas as ZIP (JlCompress is most tolerant for ZIPs)
    qDebug() << "[FW] extractPackage src:" << sourceFilePath
             << " dst:" << destinationFolderPath;

    const QStringList extracted = JlCompress::extractDir(sourceFilePath,
                                                         destinationFolderPath);

    if (!extracted.isEmpty()) {
        qDebug() << "[FW] extracted files:" << extracted.size();
        return true;
    }

    // If extractDir reported empty but files are already present, accept success.
    if (hasFilesIn(destinationFolderPath)) {
        qDebug() << "[FW] ZIP extraction empty, but destination already contains files."
                 << "Assuming already extracted.";
        return true;
    }

    // -- 3) Optional TAR fallback (very old BAS variants)
    //   Only try if header smells like USTAR to avoid pointless calls
    QFile f(sourceFilePath);
    if (f.open(QIODevice::ReadOnly)) {
        // USTAR magic is at offset 257, length 5 ("ustar")
        f.seek(257);
        const QByteArray magic = f.read(5);
        f.close();

        if (magic == "ustar") {
            qDebug() << "[FW] ZIP extraction empty, trying TAR fallback...";
            // If you have a tar/gz helper, call it here. Example:
            // bool ok = TarGzExtractor::extract(sourceFilePath, destinationFolderPath);
            // if (ok || hasFilesIn(destinationFolderPath)) return true;
        }
    }

    qDebug() << "[FW] JlCompress::extractDir returned empty for" << sourceFilePath;
    return false;
}









// In your service, right before native extract:
override suspend fun extractFirmwareFile(firmwareFileInputStream: InputStream): Outcome<SoftwareReleaseManifest> {
    val basFile = File("${context.filesDir.absolutePath}/firmware.bas")

    // Always overwrite the .bas
    basFile.outputStream().use { os ->
        firmwareFileInputStream.copyTo(os)
        // use{} guarantees close; do NOT call native before closing!
    }

    // Clean destination once per run, before the first extract
    extractionFolder.listFiles()?.forEach { it.deleteRecursively() }
    extractionFolder.mkdirs()

    // Call native extractor ONCE
    val ok = extractor.extractPackage(basFile.absolutePath, extractionFolder.absolutePath)
    if (!ok) {
        ProLog.w(MODULE_NAME, "Unable to extract bas file: ${basFile.name}")
        return Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)
    }

    // ... locate and parse manifest as you already do
    val manifestFile = extractionFolder.listFiles()?.firstOrNull { it.name.contains("manifest") }
        ?: return Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)

    val releaseManifest = try {
        ManifestParser().parse(manifestFile.inputStream())
    } catch (e: Exception) {
        ProLog.w(MODULE_NAME, "Unable to parse manifest file.")
        return Outcome.Error(e)
    }

    return Outcome.Ok(releaseManifest)
}







