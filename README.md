data class DSemVer(
    val major: Int = 0,
    val minor: Int = 0,
    val patch: Int = 0,
    val build: Int = 0
) : Comparable<DSemVer> {

    override fun compareTo(other: DSemVer): Int {
        if (major != other.major) return major.compareTo(other.major)
        if (minor != other.minor) return minor.compareTo(other.minor)
        if (patch != other.patch) return patch.compareTo(other.patch)
        return build.compareTo(other.build)
    }

    override fun toString(): String =
        if (build != 0) "$major.$minor.$patch.$build"
        else "$major.$minor.$patch"

    companion object {
        val Invalid = DSemVer()

        fun fromString(version: String): DSemVer {
            val parts = version
                .replace("[^\\d.]".toRegex(), "")
                .split(".")
                .filter { it.isNotBlank() }

            return DSemVer(
                major = parts.getOrNull(0)?.toIntOrNull() ?: 0,
                minor = parts.getOrNull(1)?.toIntOrNull() ?: 0,
                patch = parts.getOrNull(2)?.toIntOrNull() ?: 0,
                build = parts.getOrNull(3)?.toIntOrNull() ?: 0
            )
        }
    }
}











enum class DynamoYoctoOS(val yoctoPrjVer: DSemVer) {
    INVALID(DSemVer()),
    DUNFELL(DSemVer(1, 0, 0));   // example

    companion object {
        fun fromVersion(findVer: DSemVer): DynamoYoctoOS {
            if (findVer.major == 0 && findVer.minor == 0 && findVer.patch == 0) {
                return INVALID
            }

            return values().firstOrNull { yocto ->
                yocto.yoctoPrjVer.major == findVer.major &&
                yocto.yoctoPrjVer.minor == findVer.minor &&
                yocto.yoctoPrjVer.patch == findVer.patch
            } ?: INVALID
        }
    }
}







rtn = DynamoYoctoOS.fromVersion(
    DSemVer.fromString(osVer.value)
)
