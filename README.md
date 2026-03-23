Description

Extend the installer framework created in Part-1 by integrating actual application packaging and execution capabilities.
This includes bundling required artifacts (bundle.zip, launcher.exe), configuring installer workflows, and enabling execution of installation steps.

⸻

✅ Acceptance Criteria
	•	bundle.zip and launcher.exe are added to installer project
	•	Installer packaging configuration includes all required artifacts
	•	Extraction of bundled files to target installation directory is configured
	•	Installer triggers launcher execution post extraction
	•	Installation flow executes without manual intervention
	•	Dependencies (if any) are installed in correct paths
	•	Installer handles basic validation (file existence, paths, permissions)
	•	End-to-end installer run completes successfully
	•	No critical failures during installation flow

⸻

✅ Output
	•	Fully functional installer executable capable of:
	•	Packaging required artifacts
	•	Extracting bundle contents
	•	Installing dependencies
	•	Launching application via launcher.exe
	•	End-to-end installation flow validated on test system
	•	Installer ready for integration with upgrade workflow engine
