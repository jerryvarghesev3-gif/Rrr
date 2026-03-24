Description

Implement execution-level state transition from CAN-based upgrade to SFTP-based transfer within the Upgrade Workflow Engine, with version-aware decision logic.

The workflow must support a hybrid upgrade strategy:
	•	Devices below version 2.12:
	•	Must first be upgraded via CAN-based workflow
	•	Progressively upgraded until reaching minimum supported version (2.12)
	•	Devices at or above version 2.12:
	•	Can directly use SFTP-based firmware transfer

The execution engine must:
	•	Detect current device firmware version
	•	Decide upgrade path dynamically (CAN vs SFTP)
	•	Perform controlled transition from CAN → SFTP once threshold is reached
	•	Maintain correct workflow sequencing and state continuity

This ensures backward compatibility with older devices while enabling faster SFTP-based upgrades for newer versions.

⸻

Acceptance Criteria

Version-Based Decision Logic
	•	System correctly identifies device firmware version
	•	If version < 2.12:
	•	CAN-based upgrade flow is selected
	•	If version ≥ 2.12:
	•	SFTP-based upgrade flow is selected

Progressive Upgrade Handling (Legacy Devices)
	•	Devices below 2.12:
	•	Are upgraded via CAN in steps until reaching 2.12
	•	Workflow automatically re-evaluates version after each step
	•	Transition to SFTP occurs only after reaching required version

CAN Workflow Execution (Pre-2.12)
	•	CAN upgrade state machine executes correctly:
	•	Application → Bootloader
	•	Bootloader confirmation (SDO)
	•	Firmware transfer (block/segmented SDO)
	•	CRC / integrity validation
	•	Return to application mode
	•	“No heartbeat in bootloader” handled correctly

CAN → SFTP Transition
	•	Transition occurs seamlessly once version ≥ 2.12
	•	No workflow reset or loss of state
	•	Execution engine switches transport layer without failure

SFTP Execution (Post-2.12)
	•	SFTP transfer initiates successfully
	•	Firmware/data is transferred securely and completely
	•	Workflow continues using SFTP without fallback to CAN

Workflow Orchestration
	•	End-to-end workflow maintains correct sequencing across:
	•	CAN steps
	•	Transition point
	•	SFTP steps
	•	No duplicate or skipped steps

Error Handling
	•	Unsupported versions handled gracefully
	•	Failed CAN upgrade retries before transition
	•	SFTP failure triggers appropriate recovery or fallback logic

Logging & Observability
	•	Logs clearly show:
	•	Initial version detection
	•	Decision: CAN vs SFTP
	•	Upgrade progression to 2.12
	•	Transition point
	•	SFTP execution start

⸻

Output
	•	Implemented hybrid upgrade workflow engine supporting:
	•	Legacy CAN-based upgrade path
	•	Modern SFTP-based transfer
	•	Ensures:
	•	Backward compatibility for older firmware versions
	•	Optimized upgrade path for newer devices
	•	Provides:
	•	Seamless transition from CAN → SFTP based on version threshold
	•	Reliable execution across mixed-version device environments
	•	Evidence:
	•	Logs/screenshots showing:
	•	Device starting below 2.12 → upgraded via CAN
	•	Version reaching 2.12
	•	Transition to SFTP
	•	Successful completion via SFTP
	•	Executable demonstrating full hybrid workflo
