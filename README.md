EPIC 4 — Compatibility Matrix Engine

Story Points: 17 SP

Description

The Compatibility Matrix Engine is responsible for evaluating firmware compatibility, validating upgrade conditions, and determining valid upgrade paths based on predefined rules.

It processes matrix configuration files (JSON/XML), compares device firmware versions, and enforces rules such as:
	•	Version compatibility
	•	Board dependencies
	•	Upgrade constraints
	•	Transport eligibility

This engine acts as the decision-making core for upgrade planning.

⸻

Acceptance Criteria
	•	Matrix file is loaded and parsed successfully
	•	Compatibility rules are correctly interpreted
	•	Version comparison logic works for all cases
	•	Board dependency rules are validated
	•	Transport selection rules are applied
	•	Upgrade strategies are generated correctly
	•	Edge cases (invalid/missing data) handled safely

⸻

STORY 4.1 — Matrix Parser (6 SP)

Description

Parses the compatibility matrix file and converts it into an internal structure usable by the system.

⸻

Acceptance Criteria
	•	Matrix JSON schema is defined
	•	Matrix file loads without error
	•	Compatibility rules parsed correctly
	•	Version ranges interpreted correctly
	•	Invalid matrix structures are rejected
	•	Parser validated via testing

⸻

Tasks (1 SP / 1 Day Each)

Task 1 — Define Matrix JSON Schema
	•	Schema supports version, board, rules
	•	Structure documented

⸻

Task 2 — Load Matrix File
	•	File loads from AppData/package
	•	Errors handled properly

⸻

Task 3 — Parse Compatibility Rules
	•	Rules extracted correctly
	•	Rule structure stored internally

⸻

Task 4 — Parse Version Ranges
	•	Handles ranges like <2.12, >=2.12
	•	Comparison-ready format created

⸻

Task 5 — Validate Matrix Structure
	•	Invalid schema detected
	•	Error messages generated

⸻

Task 6 — Parser Developer Testing
	•	Valid + invalid cases tested
	•	No crashes/failures

⸻

STORY 4.2 — Version Comparison Engine (5 SP)

Description

Implements logic to compare firmware versions and evaluate compatibility rules based on the matrix.

⸻

Acceptance Criteria
	•	Version comparison logic works correctly
	•	Version ranges evaluated accurately
	•	Rule matching engine identifies valid rules
	•	Board dependency rules enforced
	•	Edge cases handled (missing/invalid data)

⸻

Tasks (1 SP / 1 Day Each)

Task 1 — Implement Version Comparison Logic
	•	Supports semantic/version formats
	•	Accurate comparison results

⸻

Task 2 — Implement Version Range Checks
	•	Range conditions evaluated correctly
	•	Supports multiple conditions

⸻

Task 3 — Implement Rule Matching Engine
	•	Matches device data with rules
	•	Returns applicable rule set

⸻

Task 4 — Implement Board Dependency Rules
	•	Dependencies validated
	•	Invalid combinations rejected

⸻

Task 5 — Edge Case Developer Validation
	•	Missing version
	•	Invalid format
	•	Boundary cases tested

⸻

STORY 4.3 — Upgrade Strategy Rules (6 SP)

Description

Defines how the upgrade should be executed based on compatibility results. Includes transport selection, upgrade order, and migration strategies.

⸻

Acceptance Criteria
	•	Transport method (CAN / SFTP) selected correctly
	•	Upgrade order determined correctly
	•	Migration rules applied when required
	•	Board dependencies validated before upgrade
	•	Strategy evaluation produces valid output
	•	Integration with planning engine works

⸻

Tasks (1 SP / 1 Day Each)

Task 1 — Implement Transport Selection Rules
	•	Chooses CAN or SFTP based on version/config
	•	Rules match matrix definition

⸻

Task 2 — Implement Upgrade Ordering Rules
	•	Boards ordered correctly
	•	Sequence respects dependencies

⸻

Task 3 — Implement Migration Rules
	•	Handles special upgrade paths
	•	Multi-step upgrades supported

⸻

Task 4 — Implement Board Dependency Checks
	•	Validates preconditions before upgrade
	•	Blocks invalid sequences

⸻

Task 5 — Strategy Evaluation Testing
	•	Rules produce expected outputs
	•	Complex scenarios validated

⸻

Task 6 — Integration Developer Testing
	•	Works with planning engine
	•	End-to-end validation successful

⸻

EPIC 5 — Upgrade Planning Engine

Story Points: 17 SP

Description

The Upgrade Planning Engine generates a complete upgrade execution plan based on device state, compatibility matrix, and strategy rules.

It determines:
	•	What to upgrade
	•	In what order
	•	Using which transport
	•	With what dependencies

⸻

Acceptance Criteria
	•	Devices discovered correctly
	•	Firmware versions collected
	•	Upgrade targets identified
	•	Upgrade plan generated correctly
	•	Execution sequence validated
	•	Plan simulation works without errors

⸻

STORY 5.1 — Device Discovery (6 SP)

Description

Discovers connected devices/boards and collects their configuration and firmware details.

⸻

Acceptance Criteria
	•	All connected boards detected
	•	Board configurations identified
	•	Firmware versions collected
	•	Device topology stored
	•	Invalid configurations detected

⸻

Tasks (1 SP / 1 Day Each)

Task 1 — Scan Connected Boards
	•	All devices detected
	•	Multiple boards supported

⸻

Task 2 — Identify Board Configs
	•	Board type/config extracted
	•	Data structured properly

⸻

Task 3 — Collect Firmware Versions
	•	Versions retrieved from devices
	•	Stored in model

⸻

Task 4 — Store Device Topology
	•	Relationship between boards captured
	•	Hierarchy maintained

⸻

Task 5 — Validate Device Configuration
	•	Invalid/missing configs detected
	•	Errors reported

⸻

Task 6 — Discovery Developer Testing
	•	Full scan validated
	•	Edge cases tested

⸻

STORY 5.2 — Upgrade Plan Generator (5 SP)

Description

Generates upgrade targets and steps based on compatibility evaluation.

⸻

Acceptance Criteria
	•	Versions compared with matrix
	•	Upgrade targets identified correctly
	•	Step list generated
	•	Transport method assigned
	•	Plan validated

⸻

Tasks (1 SP / 1 Day Each)

Task 1 — Compare Versions with Matrix
	•	Device vs target version evaluated

⸻

Task 2 — Identify Upgrade Targets
	•	Only required upgrades selected

⸻

Task 3 — Generate Upgrade Step List
	•	Step-by-step plan created

⸻

Task 4 — Assign Transfer Method
	•	CAN/SFTP selected per rule

⸻

Task 5 — Plan Developer Testing
	•	Plan validated for correctness

⸻

STORY 5.3 — Upgrade Sequence Builder (6 SP)

Description

Builds execution order and dependency-aware sequence for upgrade.

⸻

Acceptance Criteria
	•	Board upgrade order determined
	•	Execution queue created
	•	Dependencies handled correctly
	•	Sequence validated before execution
	•	Simulation works correctly

⸻

Tasks (1 SP / 1 Day Each)

Task 1 — Determine Board Upgrade Order
	•	Order based on dependencies

⸻

Task 2 — Create Execution Step Queue
	•	Queue generated correctly

⸻

Task 3 — Add Dependency Handling
	•	Blocks invalid execution

⸻

Task 4 — Validate Sequence Order
	•	Sequence verified before execution

⸻

Task 5 — Simulate Upgrade Execution
	•	Dry-run simulation works

⸻

Task 6 — Sequence Developer Testing
	•	End-to-end validation complete

