Description

Implement the bundle extraction and launcher installation mechanism within the installer framework.
The system should extract the packaged bundle (bundle.zip) to the target installation directory, validate extracted contents, and install/configure the launcher executable to enable application startup.

This includes handling file operations, path management, validation, and integration with the installer workflow to ensure a seamless installation experience.

⸻

✅ Acceptance Criteria

📦 Bundle Extraction
	•	bundle.zip is successfully extracted to the configured installation directory
	•	Extraction handles nested folders and file structures correctly
	•	Extraction process validates file integrity (no missing/corrupted files)
	•	Existing files are handled correctly (overwrite or clean install behavior defined)

📁 File Placement & Configuration
	•	Extracted files are placed in correct target paths
	•	Required directories are created if not present
	•	File permissions are set appropriately for execution

🚀 Launcher Installation
	•	launcher.exe is correctly placed in the installation directory
	•	Launcher is configured with required paths/configurations
	•	Launcher is executable after installation

🔗 Integration with Installer Flow
	•	Extraction and launcher setup are triggered automatically during installation
	•	Proper sequencing: extraction → validation → launcher setup
	•	Errors during extraction or setup are detected and reported

🧪 Validation
	•	Installation completes without errors
	•	Launcher can be executed successfully post-installation
	•	Application starts via launcher without manual intervention

⸻

📦 Output
	•	bundle.zip extracted successfully into target installation directory
	•	Fully structured installation folder with all required files
	•	launcher.exe installed and configured correctly
	•	Functional application launch via installed launcher
	•	Installer flow integrated with extraction and launcher setup
	•	Stable and repeatable installation behavior validated

