Description

Implement and validate device reboot handling as part of the upgrade workflow execution engine.

This task focuses on triggering a controlled device reboot at the appropriate stage in the upgrade workflow and ensuring seamless continuation of the process after the device becomes available again. The workflow engine must correctly issue the reboot command, handle temporary disconnection, and manage reconnection logic without breaking the upgrade flow.

Additionally, the system should maintain workflow state persistence across the reboot cycle and ensure that execution resumes from the correct step once the device reconnects.

⸻

Acceptance Criteria

Reboot Trigger
	•	Reboot command is successfully sent to the device at the correct workflow stage
	•	Reboot is triggered only when required based on workflow logic

Workflow Handling During Reboot
	•	Workflow engine properly pauses execution during device reboot
	•	No unintended workflow termination or state loss occurs

Device Reconnection
	•	System detects device disconnection during reboot
	•	Device reconnection is handled automatically after reboot
	•	Reconnection logic retries within defined timeout/retry limits

State Persistence
	•	Workflow state is preserved across reboot cycle
	•	Execution resumes from the correct step post-reboot

Error Handling
	•	Reboot failures or timeout scenarios are handled gracefully
	•	Proper logs/messages are generated for reboot and reconnection events

Stability
	•	Multiple reboot scenarios execute reliably
	•	No crashes or inconsistent states during reboot cycles

⸻

Output
	•	Implemented and validated device reboot handling within upgrade workflow
	•	Confirms:
	•	Successful triggering of reboot command
	•	Reliable handling of device disconnect/reconnect cycle
	•	Correct workflow continuation after reboot
	•	Provides:
	•	Robust reboot management as part of upgrade execution
	•	Seamless upgrade flow across device restarts
	•	Evidence:
	•	Logs/screenshots showing reboot trigger, disconnect, and successful reconnect
	•	Execution trace confirming workflow resumes correctly

