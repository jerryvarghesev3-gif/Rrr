Description

Implement a GUI component that lists all upgrade execution steps per board, based on detected configuration and workflow state transitions.

The screen should dynamically display:
	•	All detected boards (e.g., ICB, HIB, MCB)
	•	Execution stages per board (based on CAN upgrade state machine), such as:
	•	Application Mode
	•	Switch to Bootloader
	•	Bootloader Confirmation
	•	Bootloader Version Read
	•	Firmware Transfer (Block/Segmented SDO)
	•	CRC Validation
	•	Switch Back to Application
	•	Heartbeat Detection / Ready
	•	Firmware Verification

The GUI must:
	•	Populate board list after configuration/device discovery
	•	Show per-board execution status (not just global progress)
	•	Reflect real-time updates from backend workflow engine
	•	Indicate status using:
	•	✅ Success (tick)
	•	❌ Failure (X)
	•	⏳ In-progress
	•	Maintain correct sequencing aligned with the upgrade workflow
	•	Handle multiple boards executing sequentially or in defined order

Special behavior:
	•	Each board progresses independently through the same state machine
	•	UI must clearly show which board is:
	•	Completed
	•	In progress
	•	Failed

⸻

✅ Acceptance Criteria

Board Detection & Listing
	•	GUI displays all detected boards (ICB, HIB, MCB, etc.)
	•	Board list is populated dynamically based on configuration

Step Representation
	•	Each board shows the full list of upgrade steps aligned with CAN workflow
	•	Steps are displayed in correct execution order

Status Indicators
	•	Each step has clear status:
	•	⏳ In progress
	•	✅ Completed
	•	❌ Failed
	•	Status updates in real time based on backend events

Per-Board Execution Tracking
	•	Each board maintains independent execution status
	•	UI correctly reflects:
	•	HIB completed
	•	ICB in progress
	•	MCB pending (example scenarios)

Backend Integration
	•	GUI receives and reflects workflow state updates from execution engine
	•	No mismatch between backend state and UI

Error Handling
	•	Failed step clearly marked with ❌
	•	Failure does not corrupt display of other boards
	•	Error information is available (basic level for 1 SP)

Sequence Integrity
	•	Steps update in correct order (no skipping or misalignment)
	•	UI respects workflow sequencing rules

⸻

✅ Output
	•	GUI component that:
	•	Lists all boards involved in upgrade
	•	Displays execution steps per board
	•	Shows real-time status updates
	•	Deliverables:
	•	Working UI screen/component
	•	Screenshot showing:
	•	Multiple boards listed
	•	Mixed states (completed / in-progress / failed)
	•	Integration with backend workflow state updates

