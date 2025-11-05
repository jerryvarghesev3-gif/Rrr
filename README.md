hh



QStringList extracted;

// Try as ZIP first
extracted = JlCompress::extractDir(sourceFilePath, destinationFolderPath);
if (extracted.isEmpty()) {
    qDebug() << "[FW] ZIP extraction empty, trying TAR fallback...";
    QProcess::execute("/system/bin/sh", {"-c",
        QString("tar -xf %1 -C %2").arg(sourceFilePath).arg(destinationFolderPath)});
    QDir dir(destinationFolderPath);
    extracted = dir.entryList(QDir::Files | QDir::NoDotAndDotDot);
}

if (extracted.isEmpty()) {
    ProLog().e(MODULE_NAME,
               qPrintable(QString("Extraction failed for %1").arg(sourceFilePath)));
    return false;
}









import android.content.Context
import java.io.File
import java.io.FileInputStream
import java.io.FileOutputStream
import javax.crypto.Cipher
import javax.crypto.CipherInputStream
import javax.crypto.spec.GCMParameterSpec

private const val GCM_NONCE_LENGTH = 12
private const val GCM_TAG_LENGTH = 16
private const val ALGO = "AES/GCM/NoPadding"

fun decryptFileToTemp(context: Context, encFile: File, plainSuffix: String): File {
    // Create temporary output directory
    val tmpDir = File(context.filesDir, "tmp").apply { mkdirs() }

    // Output file path — remove .enc and append desired suffix
    val out = File(tmpDir, encFile.name.removeSuffix(".enc") + plainSuffix)

    FileInputStream(encFile).use { fin ->
        val iv = ByteArray(GCM_NONCE_LENGTH)
        require(fin.read(iv) == iv.size) { "IV missing in ${encFile.name}" }

        val cipher = Cipher.getInstance(ALGO)
        val spec = GCMParameterSpec(GCM_TAG_LENGTH * 8, iv)
        cipher.init(Cipher.DECRYPT_MODE, encryptionKey, spec)

        CipherInputStream(fin, cipher).use { cin ->
            FileOutputStream(out).use { fout ->
                val buf = ByteArray(64 * 1024)
                while (true) {
                    val n = cin.read(buf)
                    if (n <= 0) break  // EOF reached
                    fout.write(buf, 0, n)
                }
                fout.fd.sync() // Flush data to disk before closing
            }
        }
    }

    require(out.length() > 0L) { "Decrypted empty: ${out.path}" }
    return out
}
