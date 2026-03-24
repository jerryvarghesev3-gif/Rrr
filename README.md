Description

Implement the core comparison logic between device (bed) firmware versions and the manifest file to drive upgrade workflow generation.

This component reads firmware versions from the connected device and compares them against the expected versions defined in the manifest. Based on this comparison, it determines upgrade requirements and constructs the appropriate workflow logic.

The implementation ensures accurate detection of version differences, handles edge cases, and prepares a structured upgrade plan that aligns with defined rules and system configuration.

⸻

Acceptance Criteria

Data Loading & Validation
	•	Manifest file, firmware binaries, and configuration data are loaded successfully
	•	Device firmware (bed) versions are read correctly from the connected system
	•	System validates availability and integrity of required inputs before comparison

Version Comparison Logic
	•	Device firmware versions are correctly compared against manifest entries
	•	System accurately identifies:
	•	Up-to-date components
	•	Components requiring upgrade
	•	Version mismatches

Edge Case Handling
	•	Handles scenarios such as:
	•	Missing version data
	•	Unsupported or unknown versions
	•	Invalid or corrupted manifest entries

Workflow Generation
	•	Upgrade workflow is generated based on comparison results
	•	Workflow includes only required upgrade steps (no redundant actions)
	•	Correct components/boards are selected for upgrade

Execution Readiness
	•	Generated workflow is structured and consumable by upgrade execution engine
	•	Workflow state reflects actual device and manifest conditions

Stability
	•	No crashes or inconsistent behavior during comparison and workflow generation

⸻

Output
	•	Version comparison module implemented for device vs manifest validation
	•	Generates:
	•	Accurate upgrade decision (no upgrade / partial / full upgrade)
	•	Structured workflow logic based on version differences
	•	Ensures:
	•	Only required components are included in upgrade flow
	•	Workflow is ready for execution by backend engine
	•	Integrated into:
	•	Launcher startup flow
	•	Automated firmware upgrade workflow

