enum class DynamoYoctoOS(val yoctoPrjVer: DSemVer) {
    INVALID(DSemVer(0, 0, 0, 0)),

    //Dunfell is 1.0.0.0 (this is correct base)
    DUNFELL(DSemVer(1, 0, 0, 0));

    companion object {
        fun fromVersion(findVer: DSemVer): DynamoYoctoOS {
            // treat all-zero as invalid
            if (findVer.major == 0 && findVer.minor == 0 && findVer.patch == 0 && findVer.build == 0) {
                return INVALID
            }

            //match ALL 4 parts (important!)
            return values().firstOrNull { yocto ->
                yocto.yoctoPrjVer.major == findVer.major &&
                yocto.yoctoPrjVer.minor == findVer.minor &&
                yocto.yoctoPrjVer.patch == findVer.patch &&
                yocto.yoctoPrjVer.build == findVer.build
            } ?: INVALID
        }
    }
}






val somFwSemVer = state.boardStatuses[DynamoBoard.SYSTEM_ON_MODULE]?.firmwareVersion

val somFwDVer = somFwSemVer
    ?.let { DSemVer.fromString(it.toString()) }
    ?: DSemVer()   // 0.0.0.0 fallback

fwBloc.evaluateFirmwareCompatibility(
    firmwareFileName = selectedFile.name ?: "",
    firmwareFileInputStream = inputStream,
    somVersion = somFwDVer,
    isBasicMode = isBasic,
    boardsToUpdate = if (isBasic) null else state.advancedBoardList
)
