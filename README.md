Description

Implement the functionality to load, parse, and transform the compatibility matrix file into structured internal data usable by the upgrade decision engine.

This component is responsible for:
	•	Reading the matrix JSON file from the configured location (e.g., AppData/package)
	•	Validating the file structure against the defined schema
	•	Parsing device information, firmware versions, and compatibility rules
	•	Converting rules into structured internal models for efficient evaluation

The parsed output will be used by the version comparison engine and upgrade workflow to determine valid upgrade paths and compatibility constraints.

This task includes:
	•	File loading and validation
	•	Rule extraction and normalization
	•	Internal data structure creation
	•	Error handling for invalid or corrupted matrix files

This task does not include:
	•	Rule execution logic
	•	UI integration

⸻

Acceptance Criteria

Matrix File Handling
	•	Matrix JSON file is successfully loaded from configured path (AppData/package or equivalent)
	•	System handles missing file scenarios gracefully with proper error reporting

Schema Validation
	•	Matrix file is validated against defined JSON schema
	•	Invalid structures are rejected with clear error messages

Parsing & Data Extraction
	•	Device/board information is correctly extracted
	•	Firmware components (boot/app/hardware) are parsed correctly
	•	Version values and version ranges are interpreted accurately

Rule Processing
	•	Compatibility rules are:
	•	Extracted correctly from JSON
	•	Parsed into structured internal models
	•	Normalized for evaluation (conditions, constraints, actions)

Internal Representation
	•	Parsed data is stored in well-defined internal data structures
	•	Data structures are accessible by:
	•	Version comparison engine
	•	Upgrade planning logic

Error Handling
	•	Handles:
	•	Missing fields
	•	Invalid version formats
	•	Corrupt JSON
	•	Logs meaningful error messages for debugging

Performance & Stability
	•	Parsing completes within acceptable time for expected file size
	•	No crashes or memory issues during parsing

Testing
	•	Unit tests validate:
	•	Successful parsing
	•	Invalid input handling
	•	Rule extraction accuracy

⸻

Output
	•	Matrix file loading mechanism implemented
	•	Compatibility matrix successfully parsed into internal data structures
	•	Structured data available for:
	•	Version comparison engine
	•	Upgrade decision logic
	•	Compatibility rules converted into:
	•	Standardized internal representation
	•	Ready-to-evaluate rule objects
	•	Error handling implemented for:
	•	Invalid matrix files
	•	Missing or malformed data
	•	Developer-tested parsing module with validation results

⸻

Short Output (Jira-friendly)
	•	Matrix JSON file loaded, validated, and parsed into structured internal models
	•	Compatibility rules extracted and prepared for version comparison and upgrade logic

