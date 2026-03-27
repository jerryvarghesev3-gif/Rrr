📝 Description

Perform end-to-end developer testing for Security & Login, GUI landing pages, communication entry screens (CAN/SFTP), and failure mode/error handling screens.

This testing ensures that:
	•	Secure login and session handling work correctly
	•	GUI landing/entry screens behave as expected
	•	Communication-based flows (CAN/SFTP) initialize properly
	•	Failure scenarios trigger correct error screens and state transitions

The scope includes validating full user flow:
Login → Landing Page → Communication Selection (CAN/SFTP) → Workflow Execution → Failure/Error Handling → Recovery/Navigation

Testing should simulate real-world usage, including:
	•	Valid/invalid login attempts
	•	Session timeout and inactivity behavior
	•	Navigation across GUI screens
	•	Communication failures and system errors

⸻

✅ Acceptance Criteria

🔹 Security & Login
	•	Login screen is displayed as secure entry point
	•	Authentication works for valid credentials
	•	Invalid login attempts are handled with proper error messages
	•	Session is created and managed securely after login
	•	Logout and session timeout behave correctly

⸻

🔹 GUI Landing Page
	•	Landing page loads after successful login
	•	Displays correct options/features
	•	Navigation from landing page works without issues
	•	No broken or inconsistent UI states

⸻

🔹 CAN / SFTP Landing Screen
	•	User can select or access CAN/SFTP workflows
	•	Connection initialization is triggered correctly
	•	Status (connected/disconnected) is reflected on UI
	•	Proper error handling if connection fails

⸻

🔹 Failure Mode & Error Handling Screens
	•	Failure scenarios trigger appropriate error screens
	•	Error messages are clear and relevant to the issue
	•	GUI transitions correctly to failure state
	•	No application crash or undefined state

⸻

🔹 State Transitions
	•	Smooth transitions across:
	•	Login → Landing page
	•	Landing → Communication screens
	•	Workflow → Error screen (on failure)
	•	State consistency maintained between GUI and backend

⸻

🔹 End-to-End Flow Validation
	•	Complete flow works without breaking:
	•	Login → Navigation → Workflow → Failure/Recovery
	•	All modules interact correctly (GUI + backend)
	•	No deadlocks, stuck states, or unexpected behavior

⸻

🔹 Stability & Reliability
	•	System remains stable under:
	•	Multiple navigation steps
	•	Failure scenarios
	•	Invalid inputs
	•	No crashes or UI freezes

⸻

🔹 Logging & Traceability
	•	Events logged for:
	•	Login attempts
	•	Navigation steps
	•	Connection status (CAN/SFTP)
	•	Failures/errors
	•	Logs help in debugging and validation

