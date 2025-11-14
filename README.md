ll




private fun somOsVersionFromManifest(manifest: SoftwareReleaseManifest): SemVer? {
    val somComponent = manifest.components
        .firstOrNull { it.typeId == "SOM-OS" }  // or .TypeId if capitalised
    return somComponent?.version
        ?.let { SemVer.fromString(it) }
}






if (pkgOsVersion == DynamoYoctoOS.INVALID) {

    // 1) Read OS version from device and manifest
    val deviceOs = SemVer.fromString(dynamo.getOperatingSystemVersion())

    val manifestSomOs = somOsVersionFromManifest(releaseManifest)
    // If we can’t read SOM-OS version from manifest, don’t try to push OS
    if (manifestSomOs == null) {
        ProLog.e(MODULE_NAME, "No SOM-OS component found in manifest.")
        // leave pkgOsVersion as INVALID
    } else {

        // 2) Decide if OS update is needed
        val osUpdateNeeded = deviceOs != manifestSomOs

        if (osUpdateNeeded) {
            // 3) Only if versions differ, mark the SOM OS package as present
            extractionFolder.listFiles()?.forEach { file ->
                if (file.name == dynamoAppTar) {
                    pkgOsVersion = DynamoYoctoOS.DUNFELL   // or fromVersion(manifestSomOs)
                    return@forEach
                }
            }
        }
    }
}





