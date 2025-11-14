ll

if ((SemVer.fromString(dynamo.getOperatingSystemVersion()) == releaseManifest.somOsVersion())


private fun SoftwareReleaseManifest.somOsVersion(): SemVer? {
    // Find first package, then the SOM-OS firmware component
    val somComponent = this.centrellaPackages
        .firstOrNull()
        ?.centrellaComponents
        ?.firstOrNull { c ->
            c.typeId.equals("SOM-OS", ignoreCase = true) &&
            c.kind.equals("firmware", ignoreCase = true)
        }

    // Convert the component's <Version> string to SemVer
    return somComponent
        ?.version
        ?.takeIf { it.isNotBlank() }
        ?.let { SemVer.fromString(it) }
}







// Extension style, same “packages.centrellaComponents.forEach { … }” idea
private fun SoftwareReleaseManifest.somOsVersion(): SemVer? {
    // Find the SOM-OS firmware component in the manifest
    val somComponent = centrellaComponents.firstOrNull { c ->
        c.typeId.equals("SOM-OS", ignoreCase = true) &&
        c.kind.equals("firmware", ignoreCase = true)
    }

    // Convert its <Version> string (e.g. "1.0.0.1") to SemVer
    return somComponent
        ?.version
        ?.takeIf { it.isNotBlank() }
        ?.let { SemVer.fromString(it) }
}




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





