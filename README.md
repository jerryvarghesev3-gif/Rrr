pp



package com.yourpkg.encryption

import java.io.File
import java.io.FileInputStream
import java.io.FileOutputStream
import java.nio.ByteBuffer
import javax.crypto.Cipher
import javax.crypto.CipherInputStream
import javax.crypto.spec.GCMParameterSpec
import javax.crypto.spec.IvParameterSpec
import javax.crypto.spec.SecretKeySpec
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.withContext

object AESEncryption {

    enum class Mode { CBC_PKCS5, GCM } // choose what you actually use

    /**
     * Decrypts an input .enc file into output (e.g. .tar.gz).
     * If [iv] is null, tries to read the first 16 bytes of [input] as IV.
     *
     * Returns true on success, false on any failure.
     */
    suspend fun decryptFile(
        input: File,
        output: File,
        keyBytes: ByteArray = keyFromHex(DEFAULT_KEY_HEX),
        iv: ByteArray? = null,
        mode: Mode = Mode.CBC_PKCS5
    ): Boolean = withContext(Dispatchers.IO) {
        var inStream: FileInputStream? = null
        var outStream: FileOutputStream? = null
        var cipherStream: CipherInputStream? = null
        try {
            inStream = FileInputStream(input)

            val realIv: ByteArray = when {
                iv != null -> iv
                else -> {
                    // read 16-byte IV header (common packing)
                    val hdr = ByteArray(16)
                    val read = inStream.read(hdr)
                    if (read != 16) throw IllegalArgumentException("IV header not present")
                    hdr
                }
            }

            val cipher = when (mode) {
                Mode.CBC_PKCS5 -> {
                    val c = Cipher.getInstance("AES/CBC/PKCS5Padding")
                    c.init(Cipher.DECRYPT_MODE, SecretKeySpec(keyBytes, "AES"), IvParameterSpec(realIv))
                    c
                }
                Mode.GCM -> {
                    val c = Cipher.getInstance("AES/GCM/NoPadding")
                    c.init(Cipher.DECRYPT_MODE, SecretKeySpec(keyBytes, "AES"), GCMParameterSpec(128, realIv))
                    c
                }
            }

            outStream = FileOutputStream(output)
            cipherStream = CipherInputStream(inStream, cipher)

            val buf = ByteArray(64 * 1024)
            while (true) {
                val n = cipherStream.read(buf)
                if (n <= 0) break
                outStream.write(buf, 0, n)
            }
            outStream.flush()
            true
        } catch (t: Throwable) {
            // TODO: replace with your logger
            android.util.Log.e("AESEncryption", "decryptFile failed: ${t.message}", t)
            false
        } finally {
            try { cipherStream?.close() } catch (_: Throwable) {}
            try { outStream?.close() } catch (_: Throwable) {}
            try { inStream?.close() } catch (_: Throwable) {}
        }
    }

    // ---------- small helpers ----------
    private const val DEFAULT_KEY_HEX = "00112233445566778899AABBCCDDEEFF" // <-- replace!

    fun keyFromHex(hex: String): ByteArray {
        val clean = hex.trim().replace(" ", "")
        require(clean.length == 32 || clean.length == 48 || clean.length == 64) {
            "AES key must be 16/24/32 bytes (32/48/64 hex chars)"
        }
        val bb = ByteBuffer.allocate(clean.length / 2)
        var i = 0
        while (i < clean.length) {
            bb.put(((clean[i].digitToInt(16) shl 4) or clean[i + 1].digitToInt(16)).toByte())
            i += 2
        }
        return bb.array()
    }
}


