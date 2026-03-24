Description

Implement device reboot handling integrated with the CAN-based upgrade state machine for ICB / HIB / MCB boards.

This subtask ensures that reboot actions are correctly triggered and managed as part of the upgrade workflow, especially during transitions between application mode ↔ bootloader mode and after firmware transfer completion.

The reboot logic must align with the defined execution flow:
	•	Trigger reboot when switching modes (application → bootloader or back)
	•	Handle temporary loss of heartbeat during bootloader mode
	•	Detect device availability post-reboot
	•	Resume workflow execution based on correct state

The system must differentiate between expected reboot behavior (no heartbeat in bootloader) and error scenarios (no heartbeat after application mode switch timeout), ensuring robust workflow continuation without false failures.

⸻

Acceptance Criteria

Reboot Trigger & Flow Alignment
	•	Reboot command is triggered at correct stages:
	•	Switching to bootloader mode
	•	Switching back to application mode
	•	Reboot integrates seamlessly with CAN upgrade state machine flow

Bootloader Transition Handling
	•	System correctly handles no heartbeat during bootloader mode (expected behavior)
	•	Bootloader mode confirmation via SDO (e.g., 0x1000) works correctly
	•	Device state is correctly updated to “bootloader mode”

Post-Reboot / Application Mode Recovery
	•	After reboot to application mode:
	•	System waits for heartbeat detection
	•	Applies required delay before validation
	•	Reads and verifies application version successfully

Reconnection & Detection
	•	Device disconnection during reboot is detected
	•	Device reconnection is automatically handled
	•	Retry and timeout mechanisms work reliably

Workflow State Continuity
	•	Workflow execution pauses during reboot and resumes correctly
	•	No state loss during reboot cycle
	•	Execution continues from the correct step in the state machine

Error Handling
	•	“No heartbeat during bootloader” → treated as valid
	•	“No heartbeat after application switch timeout” → treated as error
	•	Proper error logs and recovery handling implemented

Stability & Logging
	•	Reboot scenarios execute reliably across multiple runs
	•	Logs clearly capture:
	•	Reboot trigger
	•	Mode transitions
	•	Heartbeat loss/recovery
	•	Workflow resume

⸻

Output
	•	Implemented reboot handling integrated with CAN upgrade workflow state machine
	•	Ensures:
	•	Correct reboot triggering during mode transitions
	•	Proper handling of heartbeat behavior across modes
	•	Reliable device reconnect and workflow continuation
	•	Provides:
	•	Stable upgrade execution across reboot cycles
	•	Accurate differentiation between expected vs error scenarios
	•	Evidence:
	•	Logs/screenshots showing:
	•	Bootloader entry (no heartbeat scenario)
	•	Firmware transfer
	•	Reboot back to application mode
	•	Heartbeat detection and version verification
	•	End-to-end workflow execution trace

