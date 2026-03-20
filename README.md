Story: Version Comparison with Manifest (WAM & Little OS)

Description

Implement a focused comparison step where the launcher validates specific firmware components (WAM and Little OS) against the versions defined in the manifest.xml.

This task ensures:
	•	accurate detection of version differences for WAM and Little OS components
	•	identification of whether an upgrade is required for these components
	•	providing input to workflow logic without generating the full workflow itself

This story is limited to comparison logic for targeted components only, not full workflow generation.

⸻

Acceptance Criteria
	•	Launcher reads WAM and Little OS version information from the device
	•	Corresponding version entries are correctly parsed from manifest.xml
	•	Versions are compared accurately between device and manifest
	•	System identifies whether:
	•	version is up-to-date
	•	upgrade is required
	•	version is missing or invalid
	•	Comparison result is structured and available for further workflow decision logic
	•	Edge cases (missing manifest entry, invalid version format) are handled without application crash
