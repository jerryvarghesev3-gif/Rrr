Description

Implement execution-level state transition within the Upgrade Execution Engine to switch from CAN-based operations to SFTP-based transfer during firmware upgrade.

During upgrade execution:
	•	initial steps (e.g., device communication, preparation, identification) are handled over CAN
	•	once conditions are met, the execution engine transitions to SFTP for firmware transfer

This story ensures the execution engine:
	•	controls the transition as part of workflow execution
	•	maintains correct step sequencing
	•	hands over execution context from CAN phase to SFTP phase without breaking the upgrade flow

Scope is limited to execution state handling, not protocol implementation.

⸻

Acceptance Criteria
	•	Execution engine identifies the correct workflow step to transition from CAN phase to SFTP phase
	•	CAN execution step is marked as completed before initiating transition
	•	Internal execution state is updated to reflect SFTP transfer phase
	•	Next execution step (SFTP transfer) is triggered automatically after transition
	•	Execution context (device info, step status) is preserved across transition
	•	If transition fails, execution engine stops progression and reports failure state
	•	Transition does not skip or duplicate any workflow steps

