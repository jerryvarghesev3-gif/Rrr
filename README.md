ll



override suspend fun evaluateFirmwareCompatibility(
    firmwareFileName: String,
    firmwareFileInputStream: InputStream,
    somVersion: SemVer,
    isBasicMode: Boolean,
    boardsToUpdate: List<DBoard>?
) {
    // Enter "loading"
    update { s -> s.copy(firmwareCompatibilityStatus = DFirmwareCompatibilityStatus.Evaluating) }

    try {
        val result: Outcome<DFirmwareCompatibilityStatus> =
            withContext(Dispatchers.IO) {
                // Hard ceiling to prevent endless spinner
                withTimeoutOrNull(30_000L) {
                    // 1) Unpack BAS and parse manifest (no decryption of *.enc here)
                    when (val mf = firmwareService.extractFirmwareFile(firmwareFileInputStream)) {
                        is Outcome.Ok -> {
                            val manifest = mf.value
                            // cache for UI/debug (you already render this in modals)
                            update { s -> s.copy(releaseManifest = manifest) }

                            // 2) Your existing logic decides the final status
                            firmwareService.evaluateFirmwareCompatibility(
                                bedStatusBloc.getSomOS(),
                                manifest
                            )
                        }
                        is Outcome.Error -> {
                            @Suppress("UNCHECKED_CAST")
                            mf as Outcome<DFirmwareCompatibilityStatus>
                        }
                    }
                } ?: Outcome.Error(DFirmwareCompatibilityStatus.Error) // timeout -> generic error
            }

        // Reflect outcome
        when (result) {
            is Outcome.Ok -> update { s -> s.copy(firmwareCompatibilityStatus = result.value) }
            is Outcome.Error -> {
                val st = (result.error as? DFirmwareCompatibilityStatus)
                    ?: DFirmwareCompatibilityStatus.Error
                update { s -> s.copy(firmwareCompatibilityStatus = st) }
            }
        }
    } catch (e: Exception) {
        ProLog.e(MODULE_NAME, "Firmware Compatibility exception = ${e}")
        update { s -> s.copy(firmwareCompatibilityStatus = DFirmwareCompatibilityStatus.Error) }
    }
}




override suspend fun extractFirmwareFile(
    firmwareFileInputStream: InputStream
): Outcome<SoftwareReleaseManifest> {

    // Persist the uploaded BAS to app storage
    val basFile = File("${context.filesDir.absolutePath}/firmware_raw.bas")
    val extractionFolder = File("${context.filesDir.absolutePath}/fw/unpacked")

    // Ensure clean extraction dir
    if (extractionFolder.exists()) {
        extractionFolder.listFiles()?.forEach { it.deleteRecursively() }
        extractionFolder.deleteRecursively()
    }
    extractionFolder.mkdirs()

    // Copy stream -> file
    basFile.outputStream().use { os ->
        firmwareFileInputStream.copyTo(os)
        os.flush()
        (os.fd).sync()     // keep this for durability on some devices
    }

    // Ask your native/helper to extract the BAS (ZIP first, TAR fallback is inside)
    val ok = dynamo.extractBasFile(
        basFile.absolutePath,
        extractionFolder.absolutePath
    )
    if (!ok) {
        ProLog.w(MODULE_NAME, "Unable to extract bas file: ${basFile.name}")
        return Outcome.Error(DFirmwareCompatibilityStatus.Error)
    }

    // Find manifest inside the extracted folder
    val manifestFile = extractionFolder
        .walkTopDown()
        .firstOrNull { it.isFile && it.name.equals("manifest.xml", ignoreCase = true) }
        ?: return Outcome.Error(DFirmwareCompatibilityStatus.Error)

    return try {
        val releaseManifest = ManifestParser().parse(manifestFile.inputStream())
        Outcome.Ok(releaseManifest)
    } catch (e: Exception) {
        ProLog.w(MODULE_NAME, "Unable to parse manifest file. ${e.message}")
        Outcome.Error(DFirmwareCompatibilityStatus.Error)
    }
}







// --- tiny helpers (keep near the service or in a file-local section) ---
private fun File.baseNameForMatch(): String =
    if (name.endsWith(".enc", ignoreCase = true)) name.removeSuffix(".enc") else name

private fun File.decryptIfEnc(context: Context): File {
    return if (name.endsWith(".enc", ignoreCase = true)) {
        // Reuse your existing decryptor. It should return a plain file.
        // Extension doesn’t matter, but we’ll prefer .tar.gz for downstream readers.
        Encryption.decryptFileToTemp(
            context = context,
            encFile = this,
            plainSuffix = ".tar.gz" // produces <cache>/.../firmware_raw_XXXX.tar.gz
        )
    } else {
        this
    }
}

// ----------------------------------------------------------------------

override suspend fun evaluateFirmwareCompatibility(
    activeOsVersion: DynamoYoctoOS,
    releaseManifest: SoftwareReleaseManifest,
): Outcome<DynamoFirmwareCompatibilityStatus> {

    var containsOS = false
    var ospkgFound = false
    var pkgOsVersion = DynamoYoctoOS.INVALID

    // 1) Look for the OS package tarball (supports .tar.gz and .tar.gz.enc)
    extractionFolder.listFiles()?.forEach { f ->
        val match = f.baseNameForMatch()
        if (match == ospkgTarGzFileName) {
            ospkgFound = true

            // decrypt to a temp file only if needed
            val readable = f.decryptIfEnc(context)

            when (val ver = Dynamo.extractYoctoVersion(context, readable)) {
                is Outcome.Ok -> {
                    lastEvaluatedFwUpdateOsVersion = SemVer.fromString(ver.value)
                    containsOS = shouldContainOS(lastEvaluatedFwUpdateOsVersion)
                    pkgOsVersion = DynamoYoctoOS.fromVersion(lastEvaluatedFwUpdateOsVersion)
                    ProLog.i(MODULE_NAME, "OS pkg detected: $match, parsed=${lastEvaluatedFwUpdateOsVersion}")
                }
                is Outcome.Error -> {
                    ProLog.e(MODULE_NAME, "Could not find yocto version in ${f.name}. Result=${ver.error}")
                    return Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)
                }
            }
            return@forEach
        }
    }

    // 2) If not found under OS name, also accept your SOM/application tarball
    if (pkgOsVersion == DynamoYoctoOS.INVALID) {
        extractionFolder.listFiles()?.forEach { f ->
            val match = f.baseNameForMatch()
            if (match == dynamoosTarGzFileName /* your second expected name */) {
                val readable = f.decryptIfEnc(context)
                when (val ver = Dynamo.extractYoctoVersion(context, readable)) {
                    is Outcome.Ok -> {
                        lastEvaluatedFwUpdateOsVersion = SemVer.fromString(ver.value)
                        containsOS = shouldContainOS(lastEvaluatedFwUpdateOsVersion)
                        pkgOsVersion = DynamoYoctoOS.fromVersion(lastEvaluatedFwUpdateOsVersion)
                        ProLog.i(MODULE_NAME, "Alt OS pkg detected: $match, parsed=${lastEvaluatedFwUpdateOsVersion}")
                    }
                    is Outcome.Error -> {
                        ProLog.e(MODULE_NAME, "Could not find yocto version in ${f.name}. Result=${ver.error}")
                        return Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)
                    }
                }
                return@forEach
            }
        }
    }

    // 3) Fallback: if still not identified, allow the “OS update not included” path
    if (pkgOsVersion == DynamoYoctoOS.INVALID) {
        extractionFolder.listFiles()?.forEach { f ->
            val match = f.baseNameForMatch()
            if (match == dynamoAppTar /* your app tar name if you use it as a signal */) {
                pkgOsVersion = DynamoYoctoOS.DUNFELL
                ProLog.i(MODULE_NAME, "No explicit OS tar found; inferred app-only payload via $match")
                return@forEach
            }
        }
    }

    // 4) Return same statuses your UI already handles
    return Outcome.Ok(
        if (pkgOsVersion == DynamoYoctoOS.INVALID) {
            DynamoFirmwareCompatibilityStatus.OSUpdateNotIncluded
        } else {
            // keep whatever you were returning before after computing 'containsOS'
            // (this line preserves your existing downstream flow)
            DynamoFirmwareCompatibilityStatus.OSUpdateIncluded
        }
    )
}
