Description

Implement GUI-based workflow management (Part 1) to control and reflect execution-level state transitions of the upgrade workflow after the board/step list is initialized.

This component acts as the UI orchestrator layer, which:
	•	Initializes workflow execution based on:
	•	Detected boards (ICB / HIB / MCB)
	•	Predefined upgrade steps (from previous ticket)
	•	Dynamically updates UI state based on backend workflow engine events aligned with the CAN upgrade state machine:
	•	Application Mode
	•	Switch to Bootloader
	•	Bootloader Confirmation
	•	Bootloader Version Read
	•	Firmware Transfer (block/segmented SDO)
	•	CRC Validation
	•	Switch back to Application
	•	Heartbeat detection
	•	Firmware version verification

The GUI must:
	•	Drive state progression visualization per board
	•	React to backend events and update:
	•	Step status (⏳ / ✅ / ❌)
	•	Active step highlight
	•	Board-level status (In-progress / Completed / Failed)
	•	Maintain synchronization between:
	•	Workflow engine state
	•	UI representation

Special handling:
	•	Bootloader phase → no heartbeat expected (UI should not flag error)
	•	Application phase → heartbeat required to mark readiness
	•	Failure at any step → UI stops progression for that board and highlights failure

This ticket ensures UI workflow state lifecycle is correctly initialized and updated, but does not include advanced controls (retry, pause — handled in later parts).

⸻

✅ Acceptance Criteria

Workflow Initialization
	•	GUI initializes workflow state after board/step list is loaded
	•	All boards start in Application Mode / Initial state

State Transition Handling
	•	UI updates correctly for each state transition received from backend
	•	Active step is clearly indicated per board
	•	Steps follow correct sequence (no skipping or disorder)

Real-Time Updates
	•	Step status updates dynamically:
	•	⏳ In progress
	•	✅ Completed
	•	❌ Failed
	•	No manual refresh required

Per-Board Execution Management
	•	Each board progresses independently
	•	UI correctly reflects mixed states:
	•	Example:
	•	HIB → Completed
	•	ICB → In progress
	•	MCB → Pending

CAN Workflow Alignment
	•	UI reflects correct behavior:
	•	No heartbeat error during bootloader
	•	Heartbeat required after switching back to application
	•	Transfer method (block/segmented SDO) does not break UI flow

Failure Handling
	•	On failure:
	•	Step is marked ❌
	•	Board execution stops
	•	UI clearly highlights failure state
	•	Other boards continue unaffected

Synchronization Integrity
	•	No mismatch between backend workflow state and UI
	•	UI transitions only when valid backend event is received

⸻

✅ Output
	•	GUI workflow management layer that:
	•	Initializes execution state
	•	Tracks real-time state transitions
	•	Updates UI per board and per step
	•	Deliverables:
	•	Working UI integrated with workflow engine
	•	Screenshot showing:
	•	Step-by-step progression
	•	Multiple boards with different states
	•	Demonstration of:
	•	Successful execution flow
	•	Failure scenario handling

