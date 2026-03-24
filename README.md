Description

Implement logic to evaluate firmware version ranges as part of the version comparison engine.

This component is responsible for interpreting and validating version constraints defined in the compatibility matrix, such as minimum, maximum, and conditional ranges. It ensures that firmware versions fall within allowed boundaries before enabling upgrade decisions.

This includes:
	•	Parsing version range definitions (e.g., min/max, inclusive/exclusive)
	•	Supporting comparison operators (>, <, >=, <=, ==)
	•	Evaluating single and multiple range conditions
	•	Integrating range checks into rule evaluation flow

This task does not include full rule execution or upgrade strategy decisions.

⸻

Acceptance Criteria

Version Range Evaluation
	•	Version ranges are evaluated correctly for:
	•	Minimum and maximum boundaries
	•	Inclusive and exclusive conditions
	•	Supports operators:
	•	Greater than (>)
	•	Less than (<)
	•	Greater than or equal (>=)
	•	Less than or equal (<=)
	•	Equal (==)

Multiple Conditions
	•	Handles multiple range conditions within a rule
	•	Correctly evaluates combined conditions (AND / OR if applicable)

Accuracy
	•	Version comparison logic works for:
	•	Standard version formats (e.g., x.y.z)
	•	Different firmware components (boot/app/hardware)

Edge Cases
	•	Handles:
	•	Invalid version formats
	•	Missing version values
	•	Boundary values (exact match with limits)

Stability
	•	No crashes during evaluation
	•	Deterministic results for same inputs

Integration Readiness
	•	Range check output is consumable by:
	•	Rule evaluation engine
	•	Upgrade decision logic

⸻

Output
	•	Version range evaluation logic implemented and validated
	•	Supports:
	•	Min/max constraints
	•	Comparison operators
	•	Multiple condition handling
	•	Accurate determination of whether a version:
	•	Falls within allowed range
	•	Violates defined constraints
	•	Output integrated into rule evaluation flow
	•	Tested module ready for use in version comparison engine

