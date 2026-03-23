Description

Develop a complete launcher GUI using Qt/QML that includes a landing page, navigation layout, and information screens. The GUI will enable users to initiate workflows, view system/device information, manage versions, and connect to external interfaces such as CAN-USB and SFTP.

The implementation includes UI design, navigation flow, backend integration, failure handling screens, and version management logic. The system should provide a seamless and user-friendly interface for interacting with the installer and upgrade workflow.

⸻

✅ Acceptance Criteria

🖥️ GUI Main Page (QT/QML)
	•	Main landing page loads successfully on launcher start
	•	UI is built using QT/QML and follows defined layout
	•	All primary UI components (buttons, sections, navigation panel) are visible and functional

⸻

🧭 Navigation Layout & Connectivity
	•	Navigation between landing page and other screens works correctly
	•	UI layout supports modular navigation (info, failure, version, connection screens)
	•	Backend connectivity is established for required operations

⸻

🔌 Info Screen (CAN-USB & SFTP)
	•	GUI allows connection to CAN-USB interface
	•	GUI supports SFTP connection configuration
	•	Connection status is displayed correctly (connected/disconnected/error)
	•	Device/system information is fetched and displayed

⸻

⚠️ Failure Mode & Error Handling Screen
	•	Failure scenarios (connection failure, communication error, etc.) are handled
	•	Error messages are displayed clearly in the GUI
	•	System transitions to failure screen without crashing
	•	User can recover or retry from failure state

⸻

🔢 Version Handling (GUI + Process)
	•	GUI displays version details (current, target, available)
	•	Version data is fetched and processed correctly
	•	Version selection or validation (if applicable) works correctly

⸻

🔄 Integration & State Management
	•	GUI reflects real-time state changes from backend
	•	Data binding between UI and backend logic works correctly
	•	No inconsistent UI states during navigation or operations

⸻

🧪 End-to-End Validation
	•	Launcher → Navigation → Info → Connection → Version → Failure handling flow works end-to-end
	•	No UI crashes, freezes, or broken transitions
	•	All screens load within acceptable time

⸻

📦 Output
	•	Fully functional QT/QML-based launcher GUI
	•	Landing page with structured navigation layout
	•	Info screen with CAN-USB & SFTP connectivity
	•	Version handling UI with correct data representation
	•	Failure handling screen with proper error messaging
	•	Integrated frontend + backend communication
	•	Stable, testable GUI ready for installer/upgrade workflow integration

