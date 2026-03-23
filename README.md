•	End-to-end dual communication foundation established for the updater, covering both CAN-based device interaction and SFTP-based firmware transfer
	•	Reliable device connectivity, identification, and version read capability enabled through CAN communication workflow
	•	Secure and controlled firmware transfer path enabled through SFTP, including authentication, progress tracking, and retry handling
	•	Communication layer prepared to support protocol switching within automated upgrade workflow based on upgrade rule and execution stage
	•	Robust communication behavior implemented with error detection, reconnection, retry, and monitoring mechanisms
	•	Reusable communication interfaces aligned with existing service tool drivers, reducing duplication while supporting automation needs
	•	Communication outputs made available to downstream modules including:
	•	launcher workflow logic
	•	upgrade planning
	•	upgrade execution
	•	GUI status and progress update
