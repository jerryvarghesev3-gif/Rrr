• Connection Rules
	•	Dynamo application connects only to Dynamo devices
	•	Centrella application connects only to Centrella devices
	•	Invalid connections display: “Wrong product connected”

• Initial Approach
	•	Attempted product identification using Application ID
	•	Did not work because both products share the same package path
\java\com\hillrom

• Second Approach
	•	Attempted product identification using device configuration
	•	Failed due to early device disconnection.
  disconnection
	•	Resulted in JNI crashes
	•	Product information could not be retrieved reliably

• Final Solution
	•	Implemented USB permission-based device detection
	•	Extracted device model number from USB information
	•	Used model number to determine product type

  • Benefits
	•	Reliable product validation
	•	Prevents wrong device connections
	•	Avoids JNI crashes
	•	Stable connection behavior
	•	Improved user experienc
