kk


// inside your DynamoFirmwareService (or the class in your screenshot)
override suspend fun extractFirmwareFile(
    firmwareFileInputStream: InputStream
): Outcome<SoftwareReleaseManifest> {

    // 1) Write input to a temp `.bas` file in app storage
    val workDir = File(context.filesDir, "fw").apply { mkdirs() }
    val rawBas = File(workDir, "firmware_raw.bas")
    FileOutputStream(rawBas).use { out -> firmwareFileInputStream.copyTo(out) }

    // 2) Detect if ZIP (encrypted containers won't start with "PK\u0003\u0004")
    fun isZip(file: File): Boolean {
        if (!file.exists() || file.length() < 4) return false
        FileInputStream(file).use { fin ->
            val sig = ByteArray(4)
            if (fin.read(sig) == 4) {
                // PK 03 04 or PK 05 06 or PK 07 08 – accept any ZIP signature
                return (sig[0] == 'P'.code.toByte() && sig[1] == 'K'.code.toByte())
            }
        }
        return false
    }

    // 3) If not a ZIP, decrypt the container → a proper .bas (ZIP)
    val basForExtract: File = if (isZip(rawBas)) {
        rawBas
    } else {
        // your existing AES-GCM routine you already added
        decryptFileToTemp(
            context = context,
            encFile  = rawBas,
            plainSuffix = ".bas"       // will create .../fw/firmware_raw.bas -> firmware_raw.bas.bas
        )
    }

    // 4) Prepare/clean extraction folder
    val extractionFolder = File(workDir, "unpacked")
    if (extractionFolder.exists()) extractionFolder.deleteRecursively()
    extractionFolder.mkdirs()

    // 5) Call native extractor (C++) with proper src + *folder* dst
    if (!dynamo.extractBasFile(basForExtract.absolutePath, extractionFolder.absolutePath)) {
        ProLog.w(MODULE_NAME, "Unable to extract bas file: ${basForExtract.name}")
        return Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)
    }

    // 6) Find manifest and parse
    val manifestFile = extractionFolder.listFiles()?.firstOrNull { it.name.contains("manifest", ignoreCase = true) }
        ?: return Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)

    val releaseManifest = try {
        ManifestParser().parse(manifestFile.inputStream())
    } catch (e: Exception) {
        ProLog.w(MODULE_NAME, "Unable to parse manifest file.")
        return Outcome.Error(e)
    }

    return Outcome.Ok(releaseManifest)
}



