Description

Validate all installation paths and ensure the launcher is correctly configured to use the extracted application files and dependencies.
This includes verifying directory paths, configuration values, and ensuring the launcher can correctly locate required resources for execution.

⸻

✅ Acceptance Criteria
	•	Installation paths (AppData / install directory) are validated and exist
	•	All required files and directories are accessible at configured paths
	•	Launcher configuration (paths, arguments, environment references) is set correctly
	•	No broken or invalid paths in configuration
	•	Launcher correctly resolves paths to application binaries and dependencies
	•	Validation errors (if any) are detected and logged
	•	Launcher is ready for execution without manual path correction

⸻

📦 Output
	•	Verified and valid installation directory paths
	•	Correctly configured launcher with proper file references
	•	No missing or misconfigured paths
	•	Launcher ready for successful execution
	•	Validation checks ensuring installation integrity

