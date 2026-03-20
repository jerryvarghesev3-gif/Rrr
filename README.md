Epic — Security & Access Management

🎯 Epic Description

Implement security mechanisms for the firmware upgrade application, including authentication, credential handling, and session/inactivity management to ensure only authorized access and safe operation.

This epic covers:
	•	user login and authentication
	•	secure credential handling
	•	session lifecycle management
	•	inactivity timeout and auto-lock

⸻

✅ Acceptance Criteria (Epic Level)
	•	User authentication is required before accessing the application
	•	Credentials are handled securely (no plain-text storage)
	•	User session is maintained after successful login
	•	Application enforces inactivity timeout and auto-lock/logout
	•	Unauthorized access is prevented
	•	Login failures and invalid attempts are handled properly
	•	Session state is cleared on logout or timeout
	•	Security behavior is consistent across all application workflows
