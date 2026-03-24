Description

Enhance the Upgrade Workflow Engine to manage and update workflow state transitions accurately across the CAN-based upgrade execution lifecycle.

The implementation must align with the defined CAN upgrade state machine, ensuring that each step in the firmware upgrade process is reflected as a valid and traceable workflow state.

The workflow state management must support:
	•	Transition from Application Mode → Bootloader Mode
	•	Bootloader confirmation via SDO (0x1000)
	•	Retrieval of bootloader version
	•	Execution of firmware transfer via:
	•	Block SDO or
	•	Segmented SDO (based on bootloader capability)
	•	Validation of firmware using CRC / data integrity checks
	•	Transition back to Application Mode
	•	Detection and handling of heartbeat signals

Additionally, the workflow must correctly handle protocol-specific behaviors:
	•	No heartbeat expected during bootloader mode (valid state)
	•	Heartbeat must resume after returning to application mode
	•	Timeout or missing heartbeat post-switch must be treated as an error

The state update mechanism should ensure:
	•	Accurate reflection of real-time execution status
	•	Synchronization between execution engine and workflow controller
	•	Clear traceability for debugging and monitoring

⸻

Acceptance Criteria

State Transition Accuracy
	•	All workflow states map correctly to CAN upgrade steps:
	•	Application → Bootloader → Transfer → Validation → Application
	•	No invalid or skipped transitions occur

Bootloader Transition Handling
	•	Command to switch to bootloader is issued successfully
	•	Bootloader entry is confirmed via SDO (0x1000)
	•	Workflow state updates to reflect bootloader mode

Firmware Transfer State Tracking
	•	Workflow correctly reflects:
	•	Transfer start
	•	Transfer progress
	•	Transfer completion
	•	Correct transport method selected:
	•	Block SDO OR segmented SDO

Data Integrity Validation
	•	CRC / integrity verification state is captured
	•	Workflow transitions to success/failure based on validation result

Heartbeat Handling
	•	“No heartbeat” during bootloader is treated as valid
	•	Heartbeat detection after switching to application mode is mandatory
	•	Missing heartbeat after timeout is flagged as error state

Error Handling & Recovery
	•	Any failure in:
	•	Mode switching
	•	Transfer
	•	Validation
results in appropriate error state update
	•	Workflow supports retry or failure termination state

Logging & Observability
	•	Each state transition is logged with:
	•	Timestamp
	•	State name
	•	Result (success/failure)
	•	Logs clearly reflect:
	•	Bootloader entry
	•	Transfer progress
	•	CRC validation
	•	Return to application mode

Synchronization
	•	Workflow state remains consistent between:
	•	Execution engine
	•	Workflow controller
	•	No mismatch between actual device state and workflow state

⸻

Output
	•	Fully implemented workflow state management aligned with CAN upgrade state machine
	•	Provides:
	•	Accurate real-time state tracking of firmware upgrade execution
	•	Robust handling of bootloader transitions, transfer, and validation
	•	Clear visibility into upgrade progress and failures
	•	Deliverables:
	•	Executable demonstrating correct state transitions
	•	Logs/screenshots showing:
	•	Full upgrade flow (Application → Bootloader → Transfer → CRC → Application)
	•	Proper handling of heartbeat behavior
	•	Error scenarios and state update
