Description

Enhance the Upgrade Workflow Engine to confirm successful firmware installation after upgrade execution, aligned with the CAN-based upgrade state machine.

This step occurs after CRC validation and switching back to Application Mode, ensuring that the firmware has been correctly installed and is actively running on the device.

The implementation must validate installation by:
	•	Detecting return to Application Mode
	•	Monitoring heartbeat restoration
	•	Reading and verifying application firmware version
	•	Processing device acknowledgment / response

Aligned with CAN upgrade flow:
	1.	Firmware transfer completes in Bootloader Mode
	2.	CRC validation confirms data integrity
	3.	System switches back to Application Mode
	4.	Engine waits for heartbeat signal
	5.	After stabilization delay:
	•	Reads application firmware version
	•	Confirms version matches expected firmware
	6.	Marks installation as successful or failed

Special handling:
	•	“No heartbeat” is valid during bootloader
	•	Heartbeat must resume after application mode switch
	•	Missing heartbeat after timeout = installation failure

The workflow must ensure that installation confirmation is accurate, validated, and traceable before completing the upgrade process.

⸻

Acceptance Criteria

Application Mode Confirmation
	•	Device successfully transitions from bootloader → application mode
	•	Workflow state reflects APPLICATION_MODE_ACTIVE

Heartbeat Validation
	•	Heartbeat is detected after switching to application mode
	•	Configurable delay is applied before validation
	•	Missing heartbeat after timeout is treated as failure state

Firmware Version Verification
	•	System reads firmware version from device after reboot
	•	Retrieved version matches expected version from:
	•	Manifest / upgrade package
	•	Version mismatch is detected and flagged

Installation Confirmation Logic
	•	Installation is marked SUCCESS only if:
	•	CRC passed
	•	Application mode is active
	•	Heartbeat is present
	•	Firmware version matches expected
	•	Any failure in above results in INSTALLATION_FAILED

Workflow State Integration
	•	Workflow includes states:
	•	SWITCH_TO_APPLICATION_MODE
	•	WAIT_FOR_HEARTBEAT
	•	READ_APP_VERSION
	•	INSTALLATION_CONFIRMED
	•	INSTALLATION_FAILED

Error Handling
	•	Failure scenarios handled:
	•	No heartbeat after timeout
	•	Version mismatch
	•	Device not responding
	•	Workflow transitions to proper failure state with reason

Logging & Traceability
	•	Logs include:
	•	Application mode switch confirmation
	•	Heartbeat detection status
	•	Firmware version read value
	•	Expected vs actual comparison
	•	Full trace of:
	•	CRC → Application Mode → Heartbeat → Version → Result

End-to-End Flow Validation
	•	Complete sequence verified:
	•	Bootloader → Transfer → CRC → Application → Heartbeat → Version Check → Confirm

⸻

Output
	•	Implemented firmware installation confirmation logic integrated into upgrade workflow engine
	•	Provides:
	•	Reliable validation that firmware is correctly installed and running
	•	Safe completion of upgrade process
	•	Alignment with CAN upgrade state machine
	•	Deliverables:
	•	Executable demonstrating:
	•	Successful installation confirmation flow
	•	Failure scenarios (no heartbeat / version mismatch)
	•	Logs/screenshots showing:
	•	Heartbeat detection
	•	Firmware version verification
	•	Final workflow state (success/failure)

