Description

Perform developer-level end-to-end testing of the Upgrade Workflow Engine, ensuring that the upgrade execution flow operates correctly across all defined states in the CAN-based upgrade state machine.

This testing validates that the execution engine correctly processes the full firmware upgrade lifecycle, including:
	•	Transition from Application Mode → Bootloader Mode
	•	Bootloader confirmation via SDO (0x1000)
	•	Bootloader version detection
	•	Firmware transfer using:
	•	Block SDO
	•	Segmented SDO (based on bootloader version)
	•	CRC / data integrity validation
	•	Transition back to Application Mode
	•	Heartbeat detection and validation
	•	Firmware version verification
	•	Final installation confirmation

Testing must ensure that the workflow engine:
	•	Executes steps in the correct sequence
	•	Handles protocol-specific behaviors (e.g., no heartbeat in bootloader)
	•	Responds correctly to success and failure scenarios
	•	Maintains accurate workflow state throughout execution

The task also validates robustness of the system against:
	•	Communication interruptions
	•	Invalid responses
	•	Partial transfers
	•	Timeout conditions

⸻

Acceptance Criteria

End-to-End Workflow Execution
	•	Complete upgrade flow executes successfully:
	•	Application → Bootloader → Transfer → CRC → Application → Heartbeat → Version Check
	•	No steps are skipped or executed out of order

Bootloader & Communication Validation
	•	Device successfully switches to bootloader mode
	•	Bootloader confirmation via SDO is verified
	•	No heartbeat during bootloader is handled as valid

Firmware Transfer Validation
	•	Firmware transfer completes successfully using correct SDO method
	•	Transfer progress and completion are reflected in workflow states

CRC Validation
	•	CRC validation is executed after transfer
	•	Success and failure scenarios are both tested
	•	Workflow responds correctly to CRC mismatch

Application Mode Transition
	•	Device switches back to application mode successfully
	•	Heartbeat resumes after transition
	•	Missing heartbeat triggers failure

Firmware Installation Confirmation
	•	Firmware version is read after upgrade
	•	Version matches expected firmware
	•	Installation marked success only if all validations pass

Error & Edge Case Handling
	•	Test scenarios include:
	•	CRC failure
	•	Transfer interruption
	•	Bootloader entry failure
	•	Missing heartbeat
	•	Version mismatch
	•	Workflow transitions to correct failure states

Workflow State Accuracy
	•	Each step updates correct workflow state:
	•	Bootloader
	•	Transfer
	•	CRC
	•	Application
	•	Completion
	•	No mismatch between actual device state and workflow state

Logging & Observability
	•	Logs clearly capture:
	•	Each state transition
	•	Communication events
	•	Errors and recovery
	•	Logs provide full traceability of upgrade execution

⸻

Output
	•	Validated end-to-end upgrade workflow execution aligned with CAN state machine
	•	Provides:
	•	Confidence in upgrade execution engine behavior
	•	Verified handling of both success and failure scenarios
	•	Assurance of correct workflow orchestration
	•	Deliverables:
	•	Executable demonstrating full upgrade flow
	•	Test logs/screenshots showing:
	•	All workflow states
	•	Successful upgrade scenario
	•	At least one failure scenario
	•	Evidence of:
	•	Bootloader transition
	•	Transfer
	•	CRC validation
	•	Application mode recovery
	•	Version confirmation

