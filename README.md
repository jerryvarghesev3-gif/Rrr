Description

Implement the core version comparison engine and rule evaluation logic to determine firmware compatibility and define upgrade strategies based on the parsed compatibility matrix.

This component acts as the decision engine of the upgrade workflow, responsible for:
	•	Comparing current device firmware versions with target versions
	•	Evaluating compatibility rules defined in the matrix
	•	Determining whether an upgrade is:
	•	Allowed
	•	Restricted
	•	Conditional
	•	Applying upgrade strategy rules such as:
	•	Version range validation
	•	Dependency handling (boot/app/hardware relationships)
	•	Upgrade sequencing and ordering

The engine consumes structured data from the matrix parser and produces decisions that drive the upgrade workflow execution.

This task includes:
	•	Version comparison logic (semantic/version range handling)
	•	Rule matching and evaluation engine
	•	Upgrade strategy determination
	•	Handling dependencies and constraints

This task does not include:
	•	GUI representation
	•	Actual firmware flashing execution

⸻

Acceptance Criteria

Version Comparison
	•	Firmware versions (boot/app/hardware) are compared accurately
	•	Supports:
	•	Exact version match
	•	Version ranges (min/max)
	•	Greater-than / less-than conditions

Rule Evaluation
	•	Compatibility rules are:
	•	Evaluated correctly based on parsed matrix data
	•	Matched against device state and target versions
	•	Rule engine correctly identifies:
	•	Valid upgrade paths
	•	Restricted or unsupported upgrades

Upgrade Strategy Logic
	•	Upgrade decision output includes:
	•	Allow / Block / Conditional status
	•	Required actions (if any)
	•	Upgrade ordering rules are applied correctly (e.g., boot → app sequence if required)

Dependency Handling
	•	Board/device-specific rules enforced correctly
	•	Interdependencies between firmware components handled properly

Edge Cases
	•	Handles:
	•	Missing version data
	•	Unsupported versions
	•	Conflicting rules

Output Consistency
	•	Engine produces deterministic results for same inputs
	•	Results are structured and consumable by workflow layer

Performance & Stability
	•	Rule evaluation executes within acceptable time
	•	No crashes or undefined behavior during evaluation

Testing
	•	Unit tests cover:
	•	Version comparison scenarios
	•	Rule matching cases
	•	Edge cases and invalid inputs

⸻

Output
	•	Version comparison engine implemented and validated
	•	Compatibility rules successfully evaluated to:
	•	Identify valid/invalid upgrade paths
	•	Enforce version constraints and dependencies
	•	Upgrade strategy logic defined, including:
	•	Upgrade eligibility
	•	Execution order
	•	Rule-based constraints
	•	Structured decision output generated for:
	•	Workflow execution
	•	Integration with upgrade modules
	•	Stable and tested rule engine ready for integration

