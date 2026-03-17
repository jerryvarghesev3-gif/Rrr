EPIC 3 — Communication Layer

Story Points: 20 SP

Description

The Communication Layer enables interaction between the firmware updater application and the target hardware devices. It provides support for multiple communication protocols including CAN (Controller Area Network) and SFTP (over SSH).

This layer is responsible for:
	•	Establishing and managing communication channels
	•	Discovering connected devices and boards
	•	Retrieving device and firmware information
	•	Transferring firmware binaries
	•	Handling communication failures, retries, and reconnection

The communication layer abstracts protocol-specific implementations and provides a common interface to higher-level modules such as compatibility evaluation, upgrade planning, and execution engines.

⸻

Acceptance Criteria
	•	CAN communication is initialized and functional.
	•	SFTP communication is established and functional.
	•	Device discovery works over CAN.
	•	Device information (firmware, hardware, serial) can be retrieved.
	•	File transfer via SFTP is successful.
	•	Communication errors are handled gracefully.
	•	Retry and reconnection mechanisms are implemented.
	•	Communication workflows are validated through developer testing.

⸻

STORY 3.1 — CAN Communication

Story Points: 8 SP

Description

This story implements CAN-based communication between the updater application and the device. It enables device discovery, identification, and firmware information retrieval through CAN protocol.

The system must support robust communication handling including error detection and reconnection.

⸻

Acceptance Criteria
	•	CAN driver initializes successfully.
	•	CAN communication channel opens correctly.
	•	Node discovery detects connected devices/boards.
	•	Device identification request returns valid response.
	•	Firmware version can be read via CAN.
	•	CAN communication errors are detected and handled.
	•	Reconnection logic works after communication failure.
	•	CAN communication workflow passes developer testing.

⸻

Tasks (1 SP / 1 Day Each)

Task 1 — Initialize CAN Driver
	•	CAN driver initializes without errors
	•	Driver is accessible by application

⸻

Task 2 — Open CAN Communication
	•	CAN channel opens successfully
	•	Communication ready state is confirmed

⸻

Task 3 — Implement Node Discovery
	•	Nodes (boards) are detected on CAN network
	•	Multiple nodes can be identified

⸻

Task 4 — Request Device Identification
	•	Device responds with identification data
	•	Data is parsed correctly

⸻

Task 5 — Read Firmware Version
	•	Firmware version is retrieved via CAN
	•	Data is stored in device model

⸻

Task 6 — Handle CAN Errors
	•	Communication errors are detected
	•	Errors are logged and handled

⸻

Task 7 — Reconnect Logic
	•	System attempts reconnection after failure
	•	Communication resumes after reconnect

⸻

Task 8 — CAN Developer Testing
	•	All CAN operations tested successfully
	•	No critical failures observed

⸻

STORY 3.2 — SFTP Communication

Story Points: 7 SP

Description

This story implements SFTP-based communication used for firmware transfer and advanced device interaction over network (Wi-Fi/SSH).

The system must support secure file transfer, authentication, and robust retry handling for reliable firmware updates.

⸻

Acceptance Criteria
	•	SSH connection is established successfully.
	•	Authentication is handled correctly.
	•	Connection monitoring detects status changes.
	•	Firmware files can be uploaded via SFTP.
	•	Transfer progress is tracked and reported.
	•	Retry logic handles failed transfers.
	•	SFTP workflow is validated through testing.

⸻

Tasks (1 SP / 1 Day Each)

Task 1 — Implement SSH Connection
	•	SSH connection established successfully
	•	Connection parameters configurable

⸻

Task 2 — Handle Authentication
	•	Authentication succeeds with valid credentials
	•	Invalid credentials handled properly

⸻

Task 3 — Implement Connection Monitoring
	•	Connection status tracked
	•	Disconnection events detected

⸻

Task 4 — Implement File Upload
	•	Firmware files uploaded successfully
	•	Transfer completes without corruption

⸻

Task 5 — Add Transfer Progress Tracking
	•	Progress percentage available
	•	GUI/backend can access progress

⸻

Task 6 — Add Retry Logic
	•	Failed transfers retry automatically
	•	Retry limit handled correctly

⸻

Task 7 — SFTP Developer Testing
	•	File transfer tested successfully
	•	Retry and monitoring validated

⸻

STORY 3.3 — Device Information Retrieval

Story Points: 5 SP

Description

This story focuses on retrieving device-specific information required for firmware upgrade decision-making. The system reads firmware version, bootloader version, hardware version, and serial number from the device.

The retrieved data is stored in the device data model and used by the compatibility matrix and upgrade planning engine.

⸻

Acceptance Criteria
	•	Firmware version is read successfully.
	•	Bootloader version is retrieved correctly.
	•	Hardware version is read correctly.
	•	Device serial number is retrieved.
	•	Retrieved data is stored in device data model.
	•	Device scan workflow passes developer validation.

⸻

Tasks (1 SP / 1 Day Each)

Task 1 — Read Firmware Version
	•	Firmware version retrieved successfully
	•	Data stored correctly

⸻

Task 2 — Read Bootloader Version
	•	Bootloader version retrieved
	•	Data stored in structure

⸻

Task 3 — Read Hardware Version
	•	Hardware version retrieved
	•	Data is accurate

⸻

Task 4 — Read Device Serial Number
	•	Serial number retrieved
	•	Unique identification verified

⸻

Task 5 — Device Scan Developer Validation
	•	Full device scan tested
	•	All parameters retrieved correctly

