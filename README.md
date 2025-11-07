ll



override suspend fun extractFirmwareFile(
    firmwareFileInputStream: InputStream
): Outcome<SoftwareReleaseManifest> {

    // 1) Write the incoming stream into a temp BAS on internal storage
    val basFile = File("${context.filesDir.absolutePath}/firmware_raw.bas")

    return try {
        // Make sure we fully copy then close. Never reuse the original stream again.
        basFile.outputStream().use { os ->
            firmwareFileInputStream.copyTo(os)
            os.flush()
        }

        // 2) Clean extraction folder (same folder you already use)
        extractionFolder.listFiles()?.forEach { it.deleteRecursively() }
        extractionFolder.mkdirs()

        // 3) Run your existing extractor on the temp BAS file
        if (!centralla.extractBasFile(basFile.absolutePath, extractionFolder.absolutePath)) {
            ProLog.w(MODULE_NAME, "Unable to extract bas file: ${basFile.name}.")
            return Outcome.Error(FirmwareCompatibilityStatus.Error)
        }

        // 4) Locate the manifest inside the extracted folder
        val manifestFile = extractionFolder
            .listFiles()
            ?.firstOrNull { it.name.contains("manifest", ignoreCase = true) }
            ?: return Outcome.Error(FirmwareCompatibilityStatus.Error)

        // 5) Parse the manifest (ensure the file stream is closed after parsing)
        val releaseManifest = manifestFile.inputStream().use {
            ManifestParser().parse(it)
        }

        Outcome.Ok(releaseManifest)

    } catch (e: Exception) {
        ProLog.e(MODULE_NAME, "extractFirmwareFile failed", e)
        Outcome.Error(e)
    }
}







override suspend fun evaluateFirmwareCompatibility(
    activeOSVersion: DYoctoOS,
    releaseManifest: SoftwareReleaseManifest
): Outcome<DFirmwareCompatibilityStatus> {

    var containsOS = false
    var osPkgFound = false
    var pkgOsVersion = DYoctoOS.INVALID

    // --- helpers (MINIMAL additions) ----------------------------------------
    fun File.isEnc() = name.endsWith(".enc", ignoreCase = true)
    fun File.decryptedIfNeeded(): File {
        if (!isEnc()) return this
        // decrypt *.enc -> write next to it as the same name without .enc
        val out = File(parentFile, name.removeSuffix(".enc"))
        if (out.exists()) out.delete()
        // TODO: swap in your real decrypt util
        // Return same out path your ExtractYoctoVersion expects (a .tar.gz)
        val ok = Encryption.decryptFile(input = this, output = out)
        if (!ok) throw IllegalArgumentException("Decryption failed for ${this.name}")
        return out
    }
    // ------------------------------------------------------------------------

    extractionFolder.listFiles()?.forEach { f: File ->
        // Accept plain or encrypted OS tar name
        if (f.name.equals(osPkgTarGzFileName, true) ||
            f.name.equals("$osPkgTarGzFileName.enc", true)
        ) {
            osPkgFound = true
            val candidate = try { f.decryptedIfNeeded() } catch (e: Exception) {
                ProLog.e(MODULE_NAME, "OS tar decrypt failed: ${f.name}", e)
                return Outcome.Error(DFirmwareCompatibilityStatus.Error)
            }

            when (val ver = ExtractYoctoVersion(context, candidate)) {
                is Outcome.Ok -> {
                    lastEvaluatedFwUpdateOsVersion = SemVer.fromString(ver.value)
                    containsOS = shouldContainOS(lastEvaluatedFwUpdateOsVersion)
                    pkgOsVersion = DYoctoOS.fromVersion(lastEvaluatedFwUpdateOsVersion)
                }
                is Outcome.Error -> {
                    ProLog.e(
                        MODULE_NAME,
                        "Could not find yocto version in ${f.name}. Result: ${ver.error}"
                    )
                    return Outcome.Error(DFirmwareCompatibilityStatus.Error)
                }
            }
            return@forEach
        }
        // Second filename you already checked in your old code (keep behavior)
        else if (f.name.equals(DosTarGzFileName, true) ||
                 f.name.equals("$DosTarGzFileName.enc", true)
        ) {
            val candidate = try { f.decryptedIfNeeded() } catch (e: Exception) {
                ProLog.e(MODULE_NAME, "DOS tar decrypt failed: ${f.name}", e)
                return Outcome.Error(DFirmwareCompatibilityStatus.Error)
            }

            when (val ver = ExtractYoctoVersion(context, candidate)) {
                is Outcome.Ok -> {
                    lastEvaluatedFwUpdateOsVersion = SemVer.fromString(ver.value)
                    containsOS = shouldContainOS(lastEvaluatedFwUpdateOsVersion)
                    pkgOsVersion = DYoctoOS.fromVersion(lastEvaluatedFwUpdateOsVersion)
                }
                is Outcome.Error -> {
                    ProLog.e(
                        MODULE_NAME,
                        "Could not find yocto version in ${f.name}. Result: ${ver.error}"
                    )
                    return Outcome.Error(DFirmwareCompatibilityStatus.Error)
                }
            }
            return@forEach
        }
    }

    // If still INVALID, fall back to your app tar check (plain or .enc)
    if (pkgOsVersion == DYoctoOS.INVALID) {
        extractionFolder.listFiles()?.forEach { f ->
            if (f.name.equals(dynamoAppTar, true) ||
                f.name.equals("$dynamoAppTar.enc", true)
            ) {
                pkgOsVersion = DYoctoOS.DUNFELL
                return@forEach
            }
        }
    }

    // Your original return kept; add OSIncluded when we actually found it
    return if (osPkgFound && containsOS)
        Outcome.Ok(DFirmwareCompatibilityStatus.OSUpdateIncluded)
    else
        Outcome.Ok(DFirmwareCompatibilityStatus.OSUpdateNotIncluded)
}

