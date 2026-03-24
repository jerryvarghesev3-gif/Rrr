Description

Define a standardized JSON schema for the compatibility matrix used in firmware version comparison and upgrade rule evaluation.

The schema will act as the foundation contract between configuration data and backend logic, ensuring that all compatibility rules, device configurations, and version constraints are structured, consistent, and machine-readable.

This task includes:
	•	Designing JSON structure for:
	•	Device/board identification
	•	Firmware components (boot/app/hardware)
	•	Version definitions and ranges
	•	Compatibility and upgrade rules
	•	Ensuring schema supports scalability for multiple devices and future extensions
	•	Documenting schema for developer usage and validation

This task does not include parsing or rule execution logic.

⸻

Acceptance Criteria
	•	JSON schema is defined and reviewed for:
	•	Device/board identification fields
	•	Firmware version fields (boot/app/hardware)
	•	Version range handling (min/max/supported)
	•	Rule definitions (conditions, constraints, actions)
	•	Schema supports:
	•	Multiple device variants
	•	Multiple firmware components
	•	Upgrade compatibility conditions
	•	Schema structure is:
	•	Clearly documented
	•	Easy to extend for future requirements
	•	Consistent and unambiguous
	•	Sample matrix JSON file is created based on schema
	•	Schema validation approach is defined (basic validation rules or guidelines)
	•	Schema is consumable by parser implementation (no ambiguity for developers)

⸻

Output
	•	Compatibility matrix JSON schema finalized and documented
	•	Sample matrix JSON file created following defined schema
	•	Standard structure established for:
	•	Device/board identification
	•	Firmware version representation
	•	Compatibility and upgrade rules
	•	Schema ready for:
	•	Parser implementation
	•	Version comparison engine integration
	•	Documentation available for developers to create and validate matrix files

⸻

Short Output (Jira-friendly)
	•	JSON schema defined for compatibility matrix including version, board, and rule structure
	•	Sample matrix file created and documented for parser consumption

