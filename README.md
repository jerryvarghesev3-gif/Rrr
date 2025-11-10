ll



// ---- hex helpers (add inside AESEncryption) ----
private fun String.hexToBytes(): ByteArray {
    // strip spaces/0x and validate
    val s = this.trim().replace(Regex("""(?i)0x|[\s_]"""), "")
    require(s.length % 2 == 0) { "Odd-length hex: $s" }

    val out = ByteArray(s.length / 2)
    var i = 0
    var j = 0
    while (i < s.length) {
        val hi = s[i].digitToInt(16)
        val lo = s[i + 1].digitToInt(16)
        out[j++] = ((hi shl 4) or lo).toByte()
        i += 2
    }
    return out
}


private data class EncParams(val key: ByteArray, val iv: ByteArray)

private fun readEncParamsFromBasHeader(bas: File): EncParams? {
    // Example: find lines like: "KEY=<hex>" and "IV=<hex>" near the top
    bas.inputStream().buffered().use { ins ->
        val br = ins.bufferedReader(Charsets.UTF_8)
        repeat(64) {                     // scan a few lines only
            val line = br.readLine() ?: return@repeat
            if (line.startsWith("KEY=", ignoreCase = true)) {
                val keyHex = line.substringAfter("KEY=").trim()
                val ivHex  = br.readLine()?.substringAfter("IV=")?.trim() ?: return null
                return EncParams(keyHex.hexToBytes(), ivHex.hexToBytes())
            }
        }
    }
    return null
}





