Description

Implement launcher installation logic that executes after successful bundle extraction.
This step ensures extracted launcher components are properly installed, configured, and made ready for execution within the upgrade environment.

⸻

✅ Acceptance Criteria
	•	Launcher installation is triggered automatically after successful bundle extraction
	•	Extracted files are correctly placed in the required installation directory
	•	Installation process completes without errors or missing dependencies
	•	Launcher executable is properly initialized and ready for use
	•	Installation status (success/failure) is returned and logged
	•	Failure during installation is detected and reported for recovery handling

⸻

🔥 (Optional stronger version if you want to impress)
	•	Launcher installation integrates seamlessly with extraction workflow ensuring smooth transition between steps
	•	Ensures consistency and integrity of installed components before proceeding to execution phase
