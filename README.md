ll




import java.io.File
import java.nio.charset.Charset
import javax.crypto.Cipher
import javax.crypto.spec.SecretKeySpec
import javax.crypto.spec.IvParameterSpec
import javax.crypto.spec.GCMParameterSpec

object AESEncryption {

    // --- Replace this with your real default/build-time key (16/24/32 bytes for AES-128/192/256)
    private const val DEFAULT_KEY_HEX = "00112233445566778899AABBCCDDEEFF"

    /** Hex -> ByteArray (fix for unresolved reference: hexToBytes) */
    private fun String.hexToBytes(): ByteArray {
        val s = trim().replace(" ", "")
        require(s.length % 2 == 0) { "Hex string must have even length" }
        val out = ByteArray(s.length / 2)
        var i = 0
        while (i < s.length) {
            out[i / 2] = ((s[i].digitToInt(16) shl 4) or s[i + 1].digitToInt(16)).toByte()
            i += 2
        }
        return out
    }

    /** If you keep the key in hex, convert once */
    fun keyFromHex(hex: String = DEFAULT_KEY_HEX): ByteArray = hex.hexToBytes()

    enum class Mode { CBC_PKCS5, GCM }

    /**
     * Minimal blocking decrypt – no coroutines (fix for: “suspension functions can be called only…”)
     * iv: 16 bytes for CBC, 12 bytes for GCM
     */
    fun decryptFileBlocking(
        input: File,
        output: File,
        keyBytes: ByteArray,
        iv: ByteArray,
        mode: Mode = Mode.CBC_PKCS5
    ): Boolean {
        require((mode == Mode.CBC_PKCS5 && iv.size == 16) || (mode == Mode.GCM && iv.size == 12)) {
            "IV length invalid for selected mode"
        }

        val secret = SecretKeySpec(keyBytes, "AES")
        val cipher = when (mode) {
            Mode.CBC_PKCS5 -> Cipher.getInstance("AES/CBC/PKCS5Padding")
            Mode.GCM       -> Cipher.getInstance("AES/GCM/NoPadding")
        }

        val paramSpec = when (mode) {
            Mode.CBC_PKCS5 -> IvParameterSpec(iv)
            Mode.GCM       -> GCMParameterSpec(128, iv)
        }
        cipher.init(Cipher.DECRYPT_MODE, secret, paramSpec)

        input.inputStream().use { fis ->
            output.outputStream().use { fos ->
                val cis = javax.crypto.CipherInputStream(fis, cipher)
                cis.copyTo(fos)
                fos.flush()
            }
        }
        return true
    }

    // --- OPTIONAL: read key/iv from .bas header if your package writes them as text lines
    private data class EncParams(val key: ByteArray, val iv: ByteArray)

    /**
     * Looks for lines like:
     *   KEY=001122...  (hex)
     *   IV =A1B2C3...  (hex)
     * near the start of the .bas file.
     * Return null if not present (then you fall back to DEFAULT_KEY_HEX / some known IV).
     */
    private fun readEncParamsFromBasHeader(bas: File): EncParams? {
        bas.inputStream().buffered().use { ins ->
            ins.reader(Charsets.UTF_8).buffered().use { br ->
                repeat(64) {           // scan only a few lines
                    val line = br.readLine() ?: return@repeat
                    if (line.startsWith("KEY=", ignoreCase = true)) {
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
     * Convenience used by your service: decrypt *.enc -> same name without .enc (next to it).
     * You can switch Mode/IV source here to match your producer.
     */
    fun decryptIfNeeded(input: File, defaultIv: ByteArray? = null): File {
        if (!input.name.endsWith(".enc", ignoreCase = true)) return input

        val output = File(input.parentFile, input.name.removeSuffix(".enc"))
        if (output.exists()) output.delete()

        val params = readEncParamsFromBasHeader(input)
        val key = params?.key ?: keyFromHex()
        val iv  = params?.iv  ?: (defaultIv ?: ByteArray(16) { 0 }) // <- replace with your real IV policy

        val ok = decryptFileBlocking(
            input  = input,
            output = output,
            keyBytes = key,
            iv = iv,
            mode = Mode.CBC_PKCS5   // or Mode.GCM if your producer uses GCM
        )
        if (!ok) throw IllegalArgumentException("decrypt failed for ${input.name}")
        return output
    }
}
