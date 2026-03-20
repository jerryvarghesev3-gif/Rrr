Subtask 1 — Authentication & Credential Validation (2 SP)

Description

Implement user authentication by validating login credentials and establishing a secure session for the application.

This subtask focuses only on:
	•	login input handling
	•	credential validation
	•	session creation

It does not include feature-level access control or UI restriction logic.

Acceptance Criteria
	•	User can enter login credentials (username/password)
	•	Credentials are validated against defined source (local/secure storage/API)
	•	Successful login creates a valid session
	•	Invalid login shows appropriate error message
	•	No access to application workflow without successful login

⸻

🔹 Subtask 2 — Role-Based Access Control & Permission Mapping (2 SP)

Description

Implement role/credential-based access mapping to determine which features are available to the logged-in user.

This subtask focuses only on:
	•	identifying user role/level
	•	mapping permissions to features
	•	exposing access control data to GUI

It does not include UI rendering or greying out controls.

Acceptance Criteria
	•	Logged-in user is assigned a role or access level
	•	Feature permissions are mapped based on role
	•	Access control data is available to GUI layer
	•	Unauthorized features are correctly identified
	•	No UI logic is implemented in this layer

⸻

🔹 Subtask 3 — UI Access Enforcement (Disable / Grey-Out Features) (1 SP)

Description

Apply UI-level enforcement by disabling or greying out features that are not permitted based on user credentials.

This subtask focuses only on:
	•	visual restriction
	•	UI behavior based on permissions

It does not include authentication or permission calculation.

Acceptance Criteria
	•	Restricted features are visually disabled or greyed out
	•	Enabled features are accessible based on permissions
	•	UI reflects access changes immediately after login
	•	No unauthorized interaction is possible through UI
	•	UI remains consistent across all screens
















Story: GUI – Login Screen & In-activity Screen Transition

Description

Implement the GUI flow and state transition between the Login Screen and the Inactivity / Session Lock Screen.

This story covers:
	•	displaying the login screen as the secured entry point
	•	moving the user into the main application after successful authentication
	•	detecting inactivity-triggered lock state
	•	transitioning from the active application state to the inactivity screen
	•	returning from inactivity screen back to the application after successful re-authentication

This story is limited to screen behavior and state transitions only. It does not cover credential validation logic, role permission mapping, or backend authentication implementation.

Acceptance Criteria
	•	Login screen is displayed before the user can access the main application
	•	Successful authentication transitions the user from Login Screen to the main application screen
	•	When configured inactivity timeout is reached, the application transitions to the Inactivity / Session Lock Screen
	•	While the inactivity screen is active, the user cannot access the main application without re-authentication
	•	Successful re-authentication from the inactivity screen returns the user to the previous application state or defined landing state
	•	Failed re-authentication keeps the user on the inactivity screen and shows an appropriate error state
	•	Screen transitions occur without broken layout, flicker, or invalid UI state
	•	Navigation between Login Screen, Main Application state, and Inactivity Screen is controlled by application state and not by manual unrestricted page switching
