ll





object AESEncryption {

    enum class Mode { CBC_PKCS5, GCM }

    // --- tiny utils ---
    fun keyFromHex(hex: String): ByteArray {
        val c = hex.trim().replace(" ", "")
        val out = ByteArray(c.length / 2)
        var i = 0
        var j = 0
        while (i < c.length) {
            out[j++] = (((c[i].digitToInt(16) shl 4) or c[i + 1].digitToInt(16))).toByte()
            i += 2
        }
        return out
    }
    fun String.hexToBytes(): ByteArray = keyFromHex(this)

    data class EncParams(val key: ByteArray, val iv: ByteArray)

    /** Reads KEY / IV from the BAS header (early lines). Return null if not found. */
    fun readEncParamsFromBasHeader(bas: File): EncParams? {
        bas.inputStream().buffered().use { ins ->
            val br = ins.reader(Charsets.UTF_8).buffered()
            repeat(64) {           // scan only first few lines
                val line = br.readLine() ?: return null
                if (line.startsWith("KEY=", ignoreCase = true)) {
                    val keyHex = line.substringAfter("KEY=").trim()
                    val ivHex  = br.readLine()?.substringAfter("IV=")?.trim() ?: return null
                    return EncParams(keyHex.hexToBytes(), ivHex.hexToBytes())
                }
                if (line.startsWith("---")) return null // header ended (optional)
            }
        }
        return null
    }

    /** Blocking decrypt: input -> output using AES CBC/PKCS5 (or GCM if you need). */
    fun decryptFileBlocking(
        input: File,
        output: File,
        keyBytes: ByteArray,
        iv: ByteArray,
        mode: Mode = Mode.CBC_PKCS5
    ): Boolean {
        require(iv.size == 16) { "IV must be 16 bytes" }

        val secret = javax.crypto.spec.SecretKeySpec(keyBytes, "AES")
        val cipher = when (mode) {
            Mode.CBC_PKCS5 -> javax.crypto.Cipher.getInstance("AES/CBC/PKCS5Padding").apply {
                init(javax.crypto.Cipher.DECRYPT_MODE, secret,
                    javax.crypto.spec.IvParameterSpec(iv))
            }
            Mode.GCM -> javax.crypto.Cipher.getInstance("AES/GCM/NoPadding").apply {
                init(javax.crypto.Cipher.DECRYPT_MODE, secret,
                    javax.crypto.spec.GCMParameterSpec(128, iv))
            }
        }

        input.inputStream().use { fis ->
            javax.crypto.CipherInputStream(fis, cipher).use { cis ->
                output.outputStream().use { fos ->
                    cis.copyTo(fos)
                }
            }
        }
        return true
    }
}


private fun File.isEnc(): Boolean = name.endsWith(".enc", ignoreCase = true)

/** Decrypt *.enc to a sibling file with the same name minus ".enc". */
private fun File.decryptedIfNeededBlocking(): File {
    if (!isEnc()) return this
    val out = File(parentFile, name.removeSuffix(".enc"))
    if (out.exists()) out.delete()
    // The caller ensures we have KEY/IV and calls decryptFileBlocking
    return out
}


// decrypt any *.enc members in-place -> same name without ".enc"
extractionFolder.listFiles()?.forEach { f ->
    if (f.name.endsWith(".enc", ignoreCase = true)) {
        val out = File(f.parentFile, f.name.removeSuffix(".enc"))  // e.g., *.tar.gz

        // Read KEY/IV from the BAS (NON-null or return early)
        val params = AESEncryption.readEncParamsFromBasHeader(basFile)
            ?: run {
                ProLog.e(MODULE_NAME, "BAS header missing KEY/IV for ${f.name}")
                return Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)
            }

        val ok = AESEncryption.decryptFileBlocking(
            input   = f,
            output  = out,            // <-- parameter name MUST be 'output'
            keyBytes = params.key,
            iv       = params.iv,
            mode     = AESEncryption.Mode.CBC_PKCS5   // keep CBC unless you truly use GCM
        )
        if (!ok) {
            ProLog.e(MODULE_NAME, "decrypt failed: ${f.name}")
            return Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)
        }
    }
}




