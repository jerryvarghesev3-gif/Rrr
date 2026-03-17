STORY 7.3 — Upgrade Progress Screen (4 SP)

Description

This story implements the UI screen that displays real-time upgrade progress to the user. It provides visibility into the ongoing upgrade process, including progress percentage, step status, and live logs.

This screen ensures users can monitor execution, detect issues early, and understand system behavior during upgrade.

⸻

Acceptance Criteria
	•	Upgrade progress bar is displayed and updates in real-time
	•	Step status table shows current and completed steps
	•	Log console displays real-time messages
	•	UI updates dynamically based on backend events
	•	No UI freeze during long operations
	•	Rendering and data updates validated successfully

⸻

Tasks (1 SP / 1 Day Each)

Task 1 — Create Upgrade Progress Bar
	•	Progress bar UI created
	•	Updates based on backend progress

⸻

Task 2 — Create Step Status Table
	•	Table shows steps (pending / running / completed / failed)
	•	Status updates dynamically

⸻

Task 3 — Create Log Output Console
	•	Console displays logs in real-time
	•	Supports scrolling and updates

⸻

Task 4 — UI Rendering Developer Test
	•	UI renders correctly under load
	•	No lag/crash during updates




STORY 7.4 — Result Screen (4 SP)

Description

This story implements the final result screen displayed after upgrade completion. It provides a summary of the upgrade process, including firmware versions, success/failure status, and a user-friendly report.

This screen ensures clear communication of outcome and traceability of upgrade results.

⸻

Acceptance Criteria
	•	Upgrade summary screen is displayed after completion
	•	Firmware version results are shown correctly
	•	Success and failure messages are clearly visible
	•	Result reflects actual execution outcome
	•	Upgrade report UI is generated
	•	Screen loads without errors

⸻

Tasks (1 SP / 1 Day Each)

Task 1 — Create Upgrade Summary Screen
	•	Summary layout created
	•	Screen loads after workflow completion

⸻

Task 2 — Show Firmware Version Results
	•	Old vs new versions displayed
	•	Data matches backend

⸻

Task 3 — Display Success / Failure Messages
	•	Clear status shown (Success / Failed / Partial)
	•	Error messages visible if failure

⸻

Task 4 — Generate Upgrade Report UI
	•	Summary report displayed
	•	Ready for export (future scope)

