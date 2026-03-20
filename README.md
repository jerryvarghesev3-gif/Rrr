Description

Implement failure management logic within the upgrade execution flow covering reset, retry, and recovery mechanisms to ensure stable and controlled upgrade behavior.

The system should:
	•	detect failure conditions during execution (timeout, communication loss, transfer error)
	•	perform a reset of the affected execution state to bring the system into a safe and known condition
	•	trigger retry logic to re-attempt the failed step without impacting completed steps
	•	execute recovery handling to resume the workflow from the correct point or transition to a safe failure state if recovery is not possible

This task ensures the upgrade process remains resilient and avoids inconsistent or partial execution states.

⸻

Acceptance Criteria

Failure Detection
	•	System detects failures such as timeout, communication error, or transfer failure during execution
	•	Failure events are captured with sufficient context for handling

Reset Handling
	•	On failure, the system resets only the affected step or communication state (not full workflow unnecessarily)
	•	Reset brings the system into a clean and stable state before retry or recovery
	•	No residual or corrupted state remains after reset

Retry Mechanism
	•	Retry is triggered automatically after reset based on defined conditions
	•	Retry attempts are limited and configurable (e.g., max retry count)
	•	Retry resumes from the failed step without re-executing completed steps
	•	Retry attempts are logged with status and count

Recovery Handling
	•	If retry succeeds, workflow continues normally from the recovered state
	•	If retry fails beyond limit, system transitions to recovery or safe failure state
	•	Recovery ensures no partial or inconsistent upgrade state is left in the system

Stability & Logging
	•	All failure, reset, retry, and recovery events are logged properly
	•	System remains stable and does not crash during failure scenarios
