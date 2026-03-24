Description

Enhance the Upgrade Workflow Engine to perform and validate firmware CRC (data integrity) as part of the CAN-based upgrade execution flow.

This task ensures that after firmware transfer (via Block SDO or Segmented SDO), the system performs a robust CRC/data integrity verification step before transitioning back to application mode.

The CRC validation must be tightly integrated into the CAN upgrade state machine, ensuring correctness of transferred firmware and safe continuation of the workflow.

The implementation must support:
	•	Execution of CRC validation immediately after firmware transfer completes
	•	Validation of transferred .hex firmware data against expected checksum
	•	Detection of:
	•	Data corruption
	•	Partial/incomplete transfer
	•	Mismatch between expected and actual CRC

Aligned with CAN upgrade flow:
	•	Firmware is transferred in Bootloader Mode
	•	No heartbeat is expected during transfer
	•	CRC validation confirms integrity before:
	•	Switching back to Application Mode
	•	Only upon successful CRC:
	•	Workflow proceeds to application mode switch
	•	On CRC failure:
	•	Workflow transitions to error/retry state

The workflow state must reflect:
	•	Transfer completed
	•	CRC validation in progress
	•	CRC success / failure
	•	Next transition decision

⸻

Acceptance Criteria

CRC Execution
	•	CRC validation is triggered automatically after firmware transfer completion
	•	CRC calculation is performed on full transferred firmware

Integrity Validation
	•	Computed CRC matches expected CRC from:
	•	Manifest / metadata / firmware definition
	•	Any mismatch is detected and flagged

Workflow State Integration
	•	Workflow includes explicit states:
	•	TRANSFER_COMPLETED
	•	CRC_VALIDATION_IN_PROGRESS
	•	CRC_SUCCESS
	•	CRC_FAILED

Success Path
	•	On successful CRC validation:
	•	Workflow transitions to Switch to Application Mode
	•	No unnecessary retries triggered
	•	System proceeds to next step (application mode + heartbeat validation)

Failure Handling
	•	On CRC failure:
	•	Workflow transitions to Error / Retry state
	•	Transfer is marked invalid
	•	Retry logic (if applicable) is triggered
	•	Failure reason is clearly logged

CAN Protocol Alignment
	•	CRC validation occurs in bootloader context before switching modes
	•	No heartbeat expected during validation (valid behavior)
	•	Workflow does NOT proceed to application mode if CRC fails

Logging & Traceability
	•	Logs include:
	•	CRC expected value
	•	CRC computed value
	•	Validation result (pass/fail)
	•	Logs clearly show:
	•	Transfer → CRC → decision flow

End-to-End Flow Validation
	•	Full sequence validated:
	•	Application Mode
	•	Bootloader Mode
	•	Firmware Transfer
	•	CRC Validation
	•	Switch Back to Application Mode

⸻

Output
	•	Implemented CRC validation integrated into upgrade workflow engine
	•	Provides:
	•	Reliable firmware integrity verification before activation
	•	Safe transition control based on CRC result
	•	Alignment with CAN upgrade state machine
	•	Deliverables:
	•	Executable demonstrating:
	•	Successful CRC validation flow
	•	CRC failure handling scenario
	•	Logs/screenshots showing:
	•	CRC calculation and comparison
	•	Workflow state transitions (Transfer → CRC → Next step)
	•	Error handling in case of corruption

