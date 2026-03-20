Story: Launcher Core Framework – Version Comparison & Workflow Initialization

Description

This story implements the core logic of the launcher application, responsible for initializing the upgrade workflow by validating system data and building execution logic.

When the launcher starts:
	•	it loads firmware binaries, manifest file (manifest.xml), and configuration data from AppData
	•	reads current device firmware versions and system configuration
	•	compares device versions against manifest definitions
	•	builds the upgrade workflow logic based on version differences and defined rules

The launcher acts as the central orchestrator, ensuring that all required inputs are valid and generating the correct execution path for the upgrade process.

This story focuses on:
	•	version comparison logic
	•	manifest parsing and validation
	•	workflow initialization based on comparison results

⸻

Acceptance Criteria

Data Loading & Initialization
	•	Launcher successfully loads required files (manifest, binaries, configuration) from AppData or defined location
	•	System validates presence and integrity of required files before proceeding
	•	Device firmware versions are read correctly from connected system

Version Comparison Logic
	•	Firmware versions from device are compared against manifest file entries
	•	Version differences are correctly identified (up-to-date / upgrade required / mismatch)
	•	Comparison logic handles edge cases (missing version, unsupported version, invalid format)

Workflow Generation
	•	Based on comparison results, launcher generates appropriate upgrade workflow logic
	•	Workflow includes correct sequence of upgrade steps for required components/boards
	•	No unnecessary upgrade steps are generated for already up-to-date components

System Validation
	•	Launcher prevents workflow execution if required data (manifest, binaries) is invalid or missing
	•	Errors are handled gracefully and logged appropriately

Execution Readiness
	•	Launcher produces a valid workflow structure that can be consumed by the upgrade execution engine
	•	Output workflow state is consistent and reflects actual system and version conditions
