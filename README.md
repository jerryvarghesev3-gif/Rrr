ll



package com.yourpkg.crypto

import java.io.File
import java.io.FileInputStream
import java.io.FileOutputStream
import java.nio.charset.Charset
import javax.crypto.Cipher
import javax.crypto.CipherInputStream
import javax.crypto.spec.GCMParameterSpec
import javax.crypto.spec.IvParameterSpec
import javax.crypto.spec.SecretKeySpec

object AESEncryption {

    enum class Mode { CBC_PKCS5, GCM }

    // ---- small utils ----
    private fun String.hexToBytes(): ByteArray {
        val s = trim().replace(" ", "")
        require(s.length % 2 == 0) { "Hex length must be even" }
        val out = ByteArray(s.length / 2)
        var i = 0
        while (i < s.length) {
            out[i / 2] = ((s[i].digitToInt(16) shl 4) or s[i + 1].digitToInt(16)).toByte()
            i += 2
        }
        return out
    }

    private data class EncParams(val key: ByteArray, val iv: ByteArray)

    /**
     * Reads KEY and IV from the first ~64 lines of the BAS (you already embed them there).
     * Example lines:
     *   KEY=00112233445566778899AABBCCDDEEFF
     *   IV =A1B2C3D4E5F60718293A4B5C6D7E8F90
     */
    fun readEncParamsFromBasHeader(bas: File): EncParams? {
        bas.inputStream().buffered().use { ins ->
            ins.reader(Charset.forName("UTF-8")).buffered().use { br ->
                repeat(64) {
                    val line = br.readLine() ?: return@repeat
                    if (line.startsWith("KEY", ignoreCase = true)) {
                        val keyHex = line.substringAfter("KEY=").trim()
                        val ivHex  = br.readLine()?.substringAfter("IV=")?.trim() ?: return null
                        return EncParams(keyHex.hexToBytes(), ivHex.hexToBytes())
                    }
                }
            }
        }
        return null
    }

    /**
     * Non-suspending, blocking decrypt to a file. Keeps memory use low by streaming.
     * - CBC needs IV size 16
     * - GCM usually uses 12 (and we set tag length to 128 bits)
     */
    fun decryptFileBlocking(
        input: File,
        output: File,
        keyBytes: ByteArray,
        iv: ByteArray,
        mode: Mode = Mode.CBC_PKCS5
    ): Boolean {
        try {
            val secret = SecretKeySpec(keyBytes, "AES")
            val transformation = when (mode) {
                Mode.CBC_PKCS5 -> "AES/CBC/PKCS5Padding"
                Mode.GCM       -> "AES/GCM/NoPadding"
            }

            if (mode == Mode.CBC_PKCS5) require(iv.size == 16) { "CBC IV must be 16 bytes" }
            if (mode == Mode.GCM)       require(iv.size == 12) { "GCM IV must be 12 bytes" }

            val cipher = Cipher.getInstance(transformation)
            if (mode == Mode.CBC_PKCS5) {
                cipher.init(Cipher.DECRYPT_MODE, secret, IvParameterSpec(iv))
            } else {
                cipher.init(Cipher.DECRYPT_MODE, secret, GCMParameterSpec(128, iv))
            }

            FileInputStream(input).use { fis ->
                CipherInputStream(fis, cipher).use { cis ->
                    FileOutputStream(output).use { fos ->
                        val buf = ByteArray(64 * 1024)
                        while (true) {
                            val n = cis.read(buf)
                            if (n <= 0) break
                            fos.write(buf, 0, n)
                        }
                        fos.flush()
                    }
                }
            }
            return true
        } catch (t: Throwable) {
            // If you have a logger, use it here
            return false
        }
    }
}




// after successful extractBasFile(...) and before manifest parse:

// Decrypt any *.enc (in-place -> write next to it without ".enc")
extractionFolder.listFiles()?.forEach { f ->
    if (f.name.endsWith(".enc", ignoreCase = true)) {
        val out = File(f.parentFile, f.name.removeSuffix(".enc")) // e.g., *.tar.gz
        // Read KEY/IV from the BAS header
        val params = AESEncryption.readEncParamsFromBasHeader(basFile)
            ?: run {
                ProLog.e(MODULE_NAME, "BAS header missing KEY/IV for ${f.name}")
                return Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)
            }

        val ok = AESEncryption.decryptFileBlocking(
            input   = f,
            output  = out,
            keyBytes= params.key,
            iv      = params.iv,
            // if you really use GCM, switch the mode here:
            mode    = AESEncryption.Mode.CBC_PKCS5
        )
        if (!ok) {
            ProLog.e(MODULE_NAME, "decrypt failed: ${f.name}")
            return Outcome.Error(DynamoFirmwareCompatibilityStatus.Error)
        }
    }
}




