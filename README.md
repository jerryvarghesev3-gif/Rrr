Description

This story manages the post-upgrade state validation along with inactivity-driven screen transitions and session control.

After the firmware upgrade completes:
	•	the system resets and reloads updated firmware versions
	•	performs rule validation using compatibility matrix and manifest data
	•	determines the upgrade status for each board

In parallel, the GUI must enforce inactivity session management:
	•	if no user interaction is detected after upgrade completion, the application transitions automatically to an Inactivity (Session Lock) Screen
	•	the inactivity state must securely lock the UI and prevent access to upgrade results without authentication
	•	upon user re-authentication, the system restores the exact upgrade result state, including progress status and board-level results

This story ensures:
	•	accurate upgrade state reflection
	•	secure session handling through inactivity
	•	seamless UI transitions between upgrade, inactivity, and login states

⸻

Acceptance Criteria

Upgrade State Handling
	•	After upgrade completion, updated firmware versions are reloaded and validated against compatibility rules
	•	The system correctly determines board-level upgrade status (success / failure / pending)
	•	Upgrade progress/result screen displays the final evaluated state automatically

Inactivity Detection & Transition
	•	System detects inactivity based on configured timeout after upgrade or during result display
	•	On timeout, UI transitions automatically to the Inactivity (Session Lock) Screen
	•	No user interaction is possible on the upgrade/result screen once inactivity screen is active

Security & Session Lock Behavior
	•	Inactivity screen requires user authentication to proceed
	•	Unauthorized access to upgrade results is blocked while in inactivity state
	•	Sensitive upgrade data is not exposed during inactivity

State Restoration After Login
	•	After successful login from inactivity screen, the application restores the correct post-upgrade state
	•	Upgrade results (status, logs, board states) remain intact and consistent
	•	No reprocessing or duplicate upgrade execution is triggered on resume

UI Transition Integrity
	•	Transitions between:
	•	Upgrade Progress → Result Screen
	•	Result Screen → Inactivity Screen
	•	Inactivity Screen → Login → Result Screen
occur without UI glitches or invalid states

