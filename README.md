Description

Implement a firmware upgrade progress bar (Part 1) in the GUI to visually represent the real-time execution progress of the upgrade workflow.

This progress bar must be driven by workflow state transitions from the upgrade execution engine and reflect progress across:
	•	Boards (ICB / HIB / MCB)
	•	Upgrade steps defined by the CAN state machine:
	•	Application Mode
	•	Switch to Bootloader
	•	Bootloader Confirmation
	•	Bootloader Version Read
	•	Firmware Transfer (.hex via SDO)
	•	CRC Validation
	•	Switch back to Application
	•	Heartbeat detection
	•	Version verification

The progress bar should:
	•	Represent overall upgrade progress (%)
	•	Update dynamically based on:
	•	Step completion events
	•	Board-level execution progress
	•	Support multi-board aggregation logic:
	•	Progress reflects combined execution across all boards
	•	Each completed step contributes to total progress
	•	Handle workflow-specific behavior:
	•	Bootloader phase → no heartbeat expected (no negative impact on progress)
	•	Application phase → progress waits for heartbeat confirmation
	•	Failure → progress stops or reflects error state

This ticket focuses on progress visualization logic and backend integration, not advanced UI features (animations, retry, etc.).

⸻

✅ Acceptance Criteria

Progress Bar Rendering
	•	Progress bar is displayed in the UI
	•	Initial state = 0% before execution starts

Dynamic Progress Updates
	•	Progress updates automatically based on backend workflow events
	•	No manual refresh required

State-Based Progress Mapping
	•	Each workflow step contributes to progress increment
	•	Progress reflects correct sequence of CAN upgrade steps
	•	No progress jump or skipping of steps

Multi-Board Handling
	•	Progress reflects combined execution across all boards
	•	Example:
	•	If multiple boards → progress increases proportionally
	•	Not marked complete until all boards finish

Real-Time Synchronization
	•	Progress updates in near real-time with execution engine
	•	UI always reflects latest workflow state

Failure Handling
	•	On failure:
	•	Progress bar stops or indicates failure state
	•	Does not continue incrementing
	•	Partial progress remains visible

Workflow Awareness
	•	Bootloader phase handled without blocking progress due to no heartbeat
	•	Progress resumes after application mode + heartbeat detection

⸻

✅ Output
	•	Functional progress bar component integrated with upgrade workflow engine
	•	Demonstrates:
	•	Real-time progress updates
	•	Accurate mapping to workflow state transitions
	•	Multi-board progress aggregation
	•	Deliverables:
	•	UI screenshot showing progress during execution
	•	Example:
	•	In-progress state
	•	Completed state
	•	Failure scenario

