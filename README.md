ll





// ---- tiny helpers (drop near this file; no other call-sites touched) ----
private fun File.isEnc(): Boolean = name.endsWith(".enc", ignoreCase = true)

/** Decrypt *.enc to a sibling file with the same name minus ".enc".
 *  Uses your existing AESEncryption (blocking) util. If you only
 *  have a suspend version, call runBlocking here or create a
 *  non-suspend wrapper.
 */
private fun File.decryptedIfNeededBlocking(): File {
    if (!isEnc()) return this
    val out = File(parentFile, name.removeSuffix(".enc"))
    if (out.exists()) out.delete()
    val ok = AESEncryption.decryptFileBlocking(
        input = this,
        output = out,
        // keep your defaults/overloads for key/iv/mode inside the util
    )
    if (!ok) throw IllegalArgumentException("Decryption failed for ${this.name}")
    return out
}
// ------------------------------------------------------------------------

// Your OLD function with the smallest possible edits.
override suspend fun evaluateFirmwareCompatibility(
    activeOSVersion: DynamoYoctoOS,
    releaseManifest: SoftwareReleaseManifest,
): Outcome<DynamoFirmwareCompatibilityStatus> = withContext(Dispatchers.IO) {
    var containsOS = false
    var osPkgFound = false
    var pkgOsVersion = DynamoYoctoOS.INVALID

    try {
        // 1) Walk the extracted files exactly like before,
        //    but transparently decrypt when a *.enc is found.

        extractionFolder.listFiles()?.forEach { f: File ->

            // Accept plain or encrypted OS package name
            if (f.name.equals(ospkgTarGzFileName, ignoreCase = true) ||
                f.name.equals("${ospkgTarGzFileName}.enc", ignoreCase = true)
            ) {
                osPkgFound = true
                val target = f.decryptedIfNeededBlocking()   // <--- minimal addition

                when (val ver = Dynamo.extractYoctoVersion(context, target)) {
                    is Outcome.Ok -> {
                        lastEvaluatedFwUpdateOsVersion = SemVer.fromString(ver.value)
                        containsOS = shouldContainOS(lastEvaluatedFwUpdateOsVersion)
                        pkgOsVersion = DynamoYoctoOS.fromVersion(lastEvaluatedFwUpdateOsVersion)
                    }
                    is Outcome.Error -> {
                        ProLog.e(MODULE_NAME,
                            "Could not find yocto version in ${f.name}. Result: ${ver.error}")
                        return@withContext Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)
                    }
                }
                return@forEach
            }

            // Accept plain or encrypted DOS package name
            if (f.name.equals(dOsTarGzFileName, ignoreCase = true) ||
                f.name.equals("${dOsTarGzFileName}.enc", ignoreCase = true)
            ) {
                val target = f.decryptedIfNeededBlocking()   // <--- minimal addition

                when (val ver = Dynamo.extractYoctoVersion(context, target)) {
                    is Outcome.Ok -> {
                        lastEvaluatedFwUpdateOsVersion = SemVer.fromString(ver.value)
                        containsOS = shouldContainOS(lastEvaluatedFwUpdateOsVersion)
                        pkgOsVersion = DynamoYoctoOS.fromVersion(lastEvaluatedFwUpdateOsVersion)
                    }
                    is Outcome.Error -> {
                        ProLog.e(MODULE_NAME,
                            "Could not find yocto version in ${f.name}. Result: ${ver.error}")
                        return@withContext Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)
                    }
                }
                return@forEach
            }
        }

        // 2) Your old fallback: if we never found an explicit OS tar,
        //    but we see the app tar, mark it as DUNFELL (unchanged)
        if (pkgOsVersion == DynamoYoctoOS.INVALID) {
            extractionFolder.listFiles()?.forEach { f ->
                if (f.name.equals(dynamoAppTar, ignoreCase = true) ||
                    f.name.equals("${dynamoAppTar}.enc", ignoreCase = true)
                ) {
                    pkgOsVersion = DynamoYoctoOS.DUNFELL
                    return@forEach
                }
            }
        }

        // 3) Final result is unchanged from your old code
        Outcome.Ok(
            if (containsOS) DynamoFirmwareCompatibilityStatus.OSUpdateIncluded
            else DynamoFirmwareCompatibilityStatus.OSUpdateNotIncluded
        )
    } catch (e: Exception) {
        ProLog.e(MODULE_NAME, "evaluateFirmwareCompatibility failed", e)
        Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)
    }
}
