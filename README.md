Description

Create the required AppData folder structure for the application and copy extracted bundle files into the appropriate directories.
Ensure proper organization of application files and dependencies to support correct execution of the launcher and installer workflow.

⸻

✅ Acceptance Criteria
	•	AppData directory is created successfully (if not already present)
	•	Required subfolders (e.g., app, config, dependencies) are created correctly
	•	Bundle files are copied to their respective directories in AppData
	•	File structure matches expected application layout
	•	Existing files are handled correctly (overwrite or skip as defined)
	•	File copy operation completes without errors
	•	Paths are resolved correctly across environments

⸻

📦 Output
	•	AppData folder structure created with proper hierarchy
	•	Bundle files successfully copied into AppData locations
	•	Dependencies organized in correct directories
	•	File system ready for launcher execution
	•	Verified directory structure supporting application runtime

