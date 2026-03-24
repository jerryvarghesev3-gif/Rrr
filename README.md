Description

Implement a GUI framework for the Upgrade Progress Screen that provides real-time visibility into the firmware upgrade execution driven by the Upgrade Workflow Engine and CAN-based state machine.

The UI must reflect the full upgrade lifecycle, mapping backend workflow states to clear, user-visible progress indicators.

The screen should dynamically track and display the following execution stages:
	•	Application Mode (initial state)
	•	Switch to Bootloader command
	•	Bootloader confirmation (SDO 0x1000)
	•	Bootloader version detection
	•	Firmware transfer:
	•	Block SDO / Segmented SDO (based on bootloader capability)
	•	Transfer progress (percentage-based)
	•	CRC / data integrity validation
	•	Switch back to Application Mode
	•	Heartbeat detection and readiness
	•	Firmware version verification
	•	Upgrade completion / failure

The GUI framework must:
	•	Subscribe to backend workflow events (event-driven updates)
	•	Map workflow states to UI components (progress bar, stepper, logs)
	•	Handle state transitions, delays, and protocol-specific behaviors
	•	Provide live logs and status updates for traceability
	•	Handle both success and failure paths gracefully

Special considerations:
	•	No heartbeat during bootloader → UI should not show error
	•	Heartbeat expected after application switch → UI must validate readiness
	•	Transfer method selection (block/segmented SDO) must be reflected in UI
	•	CRC validation and failures must be clearly visible

⸻

Acceptance Criteria

Real-Time Progress Visualization
	•	Progress bar reflects overall upgrade percentage
	•	Stepper (or equivalent UI) shows each upgrade stage clearly
	•	Progress updates continuously during firmware transfer

State-to-UI Mapping
	•	Each backend workflow state maps correctly to UI state:
	•	Application → Bootloader → Transfer → CRC → Application → Complete
	•	No mismatch between backend state and UI display

Bootloader Handling
	•	UI shows transition to bootloader mode
	•	No heartbeat is not treated as error during bootloader phase
	•	Bootloader confirmation step is displayed

Firmware Transfer Visibility
	•	UI shows:
	•	Transfer start
	•	Transfer progress (%)
	•	Transfer method (block/segmented if applicable)
	•	Long transfers update without freezing UI

CRC Validation Feedback
	•	CRC validation step is displayed
	•	Success → proceed to next step
	•	Failure → UI shows error and stops workflow

Application Mode Recovery
	•	UI shows switch back to application mode
	•	Heartbeat detection triggers “Device Ready” state
	•	Timeout or missing heartbeat shows failure

Firmware Version Verification
	•	UI displays version check step
	•	Shows expected vs actual version (if applicable)
	•	Failure is clearly indicated

Error Handling & UX
	•	Errors are clearly displayed with:
	•	Step where failure occurred
	•	Reason (timeout, CRC fail, communication error, etc.)
	•	UI supports retry / restart (if applicable)
	•	No silent failures

Logging & Debug Visibility
	•	Live logs panel shows:
	•	Commands sent
	•	Responses received
	•	State transitions
	•	Logs update in real time

Performance & Responsiveness
	•	UI remains responsive during entire upgrade
	•	No blocking during long operations (e.g., transfer)

⸻

Output
	•	Fully functional Upgrade Progress Screen UI integrated with workflow engine
	•	Provides:
	•	Real-time visibility into upgrade execution
	•	Clear mapping of CAN upgrade state machine to UI
	•	Improved debugging and user confidence
	•	Deliverables:
	•	GUI screen implementation (progress bar + stepper + logs)
	•	Screenshots / demo showing:
	•	Full successful upgrade flow
	•	At least one failure scenario (e.g., CRC fail)
	•	Evidence of:
	•	Real-time updates
	•	Correct state transitions
	•	Error handling

