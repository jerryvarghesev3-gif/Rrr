ll



val bedOs = SemVer.fromString(dynamo.getOperatingSystemVersion())
val pkgOs = releaseManifest.somOsVersion()

// If versions MATCH → skip OS package
val shouldSendOS = bedOs != pkgOs



packages.centrellaComponents.forEach { component ->

    // Skip sending IMX OS file if version is same
    if (!shouldSendOS && component.fileName == DynamoosTarGzFileName) {
        // Skip this one
        return@forEach
    }

    // Continue transferring all other files
    val file = File(extractionFolder, component.fileName)
    if (!file.exists()) {
        cancel(FirmwareUpdateError.GENERAL_TRANSFER_ERROR)
        return@channelFlow
    }

    // send file...
}




private fun SoftwareReleaseManifest.somOsVersion(): SemVer? {
    val somComponent = this.centrellaPackages
        .flatMap { it.centrellaComponents }
        .firstOrNull { comp ->
            comp.fileName.contains("dynamo", ignoreCase = true) &&
            comp.fileName.contains("tar.gz", ignoreCase = true)
        }

    return somComponent?.version?.let { SemVer.fromString(it) }
}





val currentOs = SemVer.fromString(dynamo.getOperatingSystemVersion())
val updateOs = releaseManifest.somOsVersion()

val shouldSendOsUpdate = currentOs != updateOs




packages.centrellaComponents.forEach { comp ->

    // Skip OS update file when equal
    if (!shouldSendOsUpdate && comp.fileName == DynamoosTarGzFileName) {
        return@forEach
    }

    // normal transfer
}


