Description

Implement and validate boards execution workflow management for both unencrypted and encrypted upgrade execution, aligned with the CAN upgrade state machine (ICB / HIB / MCB).

This subtask focuses on orchestrating firmware upgrade execution across multiple boards while ensuring:
	•	Correct workflow sequencing
	•	Support for encrypted and unencrypted transfers
	•	Synchronization with transfer engine and rule-based decision logic
	•	Alignment with bootloader ↔ application mode transitions

The workflow engine must:
	•	Execute upgrade steps per board based on dependency and ordering rules
	•	Select correct transport mode (block SDO / segmented SDO)
	•	Handle encryption-aware transfer flows (secure payload handling, validation)
	•	Integrate with reboot, heartbeat detection, and state transitions
	•	Ensure consistent execution across all board types (ICB / HIB / MCB)

⸻

Acceptance Criteria

Workflow Orchestration
	•	Boards are executed in the correct sequence based on upgrade rules
	•	Workflow respects inter-board dependencies and upgrade ordering
	•	No redundant steps are executed for already up-to-date components

Encrypted / Unencrypted Execution
	•	System correctly supports both:
	•	Encrypted upgrade flow
	•	Unencrypted upgrade flow
	•	Encryption-aware execution is triggered based on configuration/rules
	•	Secure data transfer is handled without impacting workflow execution

Transfer Logic Integration
	•	Workflow correctly integrates with transfer engine:
	•	Block SDO / Segmented SDO selection based on bootloader version
	•	Transfer logic aligns with rule engine decisions

CAN State Machine Alignment
	•	Workflow follows defined execution steps:
	•	Application → Bootloader transition
	•	Bootloader confirmation (SDO)
	•	Firmware transfer (.hex)
	•	CRC / integrity validation
	•	Switch back to application mode
	•	“No heartbeat in bootloader” handled as expected behavior
	•	Heartbeat detection after application switch is validated

Decision Engine Integration
	•	Workflow decisions are driven by:
	•	Compatibility matrix
	•	Version comparison results
	•	Upgrade strategy rules
	•	Correct execution path is generated dynamically

Error Handling & Recovery
	•	Failures in transfer, encryption, or validation are handled gracefully
	•	Retry / fallback mechanisms are implemented where applicable
	•	Workflow stops or recovers appropriately based on error type

Execution Consistency
	•	Workflow produces consistent results across multiple runs
	•	Supports all board types (ICB / HIB / MCB) without deviation

Logging & Observability
	•	Logs include:
	•	Board execution order
	•	Encryption vs non-encryption path
	•	Transfer mode selection
	•	State transitions and validation results

⸻

Output
	•	Fully implemented upgrade workflow execution engine supporting:
	•	Multi-board orchestration
	•	Encrypted and unencrypted firmware transfer
	•	Rule-driven execution logic
	•	Ensures:
	•	Correct sequencing and dependency handling
	•	Secure and reliable data transfer
	•	Alignment with CAN upgrade state machine
	•	Provides:
	•	Deterministic upgrade execution across all boards
	•	Accurate workflow decisions based on system state and rules
	•	Evidence:
	•	Execution logs showing:
	•	Board order and workflow progression
	•	Encryption-aware transfer execution
	•	State transitions (bootloader ↔ application)
	•	Screenshots / logs of successful end-to-end upgrade workflow
	•	Executable (exe) demonstrating full workflow behavior

