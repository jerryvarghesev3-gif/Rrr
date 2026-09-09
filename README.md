PATENT IDEA / INVENTION DISCLOSURE

1. Title of the Invention

RAISE – Runtime Autonomous Integrity and Self-Healing Engine for Distributed Embedded Systems

Technical Title

Autonomous Runtime Health Monitoring, Configuration Integrity Verification, Root-Cause Isolation and Hierarchical Self-Healing Framework for Distributed Embedded Systems

⸻

2. Short Description

The proposed invention is a reusable embedded software framework called RAISE (Runtime Autonomous Integrity and Self-Healing Engine) that continuously monitors the runtime health of a distributed embedded system, identifies abnormal behavior and probable root causes, automatically performs an appropriate recovery action, validates whether the recovery was successful, and escalates the recovery level when required.

The framework is designed for embedded products consisting of a Linux-based System-on-Module (SOM), a Master Controller and multiple subordinate controllers connected through communication interfaces such as RPCA/RPC and CAN.

Unlike a conventional watchdog that primarily detects a system hang and resets a device, RAISE performs fault detection, dependency-aware fault correlation, root-cause isolation, configuration integrity verification, hierarchical recovery and post-recovery validation.

The framework is designed to operate locally on the embedded product without requiring AI, cloud connectivity or external human intervention for normal recovery operations.

⸻

3. Technical Field

The invention relates to:

* Embedded systems
* Embedded Linux
* Distributed embedded control systems
* Runtime software monitoring
* Configuration integrity verification
* Communication fault detection
* Root-cause isolation
* Autonomous fault recovery
* Self-healing embedded systems
* CAN-based distributed controllers
* RPC/RPCA communication
* Software service supervision
* Hardware and software health monitoring

⸻

4. Existing System / Problem

Modern embedded products often contain multiple interconnected software and hardware components.

For example:

SOM → Master Board → CAN → Controller A/B/C

The SOM may perform:

* Wi-Fi management
* Network communication
* MQTT communication
* Server communication
* Configuration processing
* Application/service execution
* RPCA/RPC communication with the Master Board

The Master Board may communicate with multiple subordinate controllers through CAN.

In a conventional architecture, when a failure occurs, the system may only report a generic error such as:

* MQTT disconnected
* Wi-Fi disconnected
* Server unavailable
* RPCA timeout
* CAN communication failure
* Controller not responding
* Application crashed

The actual root cause may exist in another dependent component.

Consequently, troubleshooting often requires:

1. Collecting logs
2. Connecting to the product
3. Inspecting configuration files
4. Checking communication interfaces
5. Restarting services
6. Rebooting the product
7. Manually identifying the actual cause

This increases service effort, product downtime and diagnostic complexity.

⸻

5. Configuration-Related Problem

A particularly difficult class of failures can occur during configuration processing.

For example, a product may receive a configuration file:

config.json

containing multiple parameters such as:

* Bootstrap credentials
* Facility credentials
* MQTT parameters
* Network parameters
* Product configuration
* Server information

The software may transform the input configuration into another runtime configuration file:

config.json → Configuration Processing → mqtt.json

The MQTT service then uses mqtt.json.

In some products, an MQTT connection failure may occur because:

* Username is empty
* Password field is missing
* MQTT server address is missing
* Incorrect field mapping occurred
* Configuration transformation failed
* Partial configuration was generated
* Configuration file was corrupted
* Configuration version is incompatible
* Process terminated during configuration generation
* File write operation was interrupted
* Unexpected reboot occurred
* Default value was incorrectly applied
* Software transformation logic produced an incorrect result

The visible symptom is:

MQTT connection/authentication failure

However, the actual root cause may be:

Configuration transformation integrity failure

A conventional monitoring system may only report the MQTT failure.

RAISE is intended to correlate these events and identify the more probable underlying cause.

⸻

6. Proposed Solution

RAISE introduces an autonomous runtime health and self-healing layer within the embedded product.

The core operating sequence is:

DETECT → CORRELATE → DIAGNOSE → RECOVER → VALIDATE → ESCALATE

The framework continuously observes system components and their dependencies.

When a failure occurs, the framework does not immediately reboot the entire product.

Instead, it attempts to:

1. Detect the failure
2. Identify related symptoms
3. Determine the affected component
4. Analyze component dependencies
5. Identify a probable root cause
6. Select the minimum-impact recovery action
7. Execute the recovery
8. Validate functional recovery
9. Escalate if recovery fails
10. Record the complete fault and recovery history

⸻

7. System Architecture

A representative architecture is:

                    ┌─────────────────────────────┐
                    │           SOM               │
                    │     Embedded Linux          │
                    │                             │
                    │  ┌───────────────────────┐  │
                    │  │        RAISE          │  │
                    │  │ Runtime Health Engine │  │
                    │  └───────────────────────┘  │
                    │            │                │
                    │     ┌──────┴──────┐         │
                    │     │             │         │
                    │   Wi-Fi        MQTT         │
                    │     │             │         │
                    │     └──────┬──────┘         │
                    │            │                │
                    │       Configuration         │
                    │        Integrity            │
                    │            │                │
                    └────────────┼────────────────┘
                                 │
                              RPCA/RPC
                                 │
                    ┌────────────▼────────────┐
                    │      MASTER BOARD       │
                    │     Health Monitoring   │
                    └────────────┬────────────┘
                                 │
                                CAN
                 ┌───────────────┼───────────────┐
                 │               │               │
          ┌──────▼─────┐  ┌──────▼─────┐  ┌──────▼─────┐
          │ Controller │  │ Controller │  │ Controller │
          │     A      │  │     B      │  │     C      │
          └────────────┘  └────────────┘  └────────────┘

The RAISE engine may reside primarily on the SOM and communicate with product-specific hardware and software components through defined interfaces.

⸻

8. Runtime Health Monitoring

RAISE continuously monitors the health of:

* SOM operating system
* Application processes
* Software services
* Wi-Fi interface
* Network interface
* MQTT connection
* Server/network reachability
* Configuration files
* Configuration transformation
* RPCA/RPC communication
* Master Board
* CAN communication
* Individual CAN nodes
* Hardware I/O
* Watchdog/runtime indicators

Each component can expose one or more health indicators such as:

* Heartbeat
* Response time
* Timeout
* Connection state
* Process state
* Configuration validity
* Communication status
* Expected response
* Error count
* Recovery history

⸻

9. Configuration Integrity Manager

RAISE includes a configuration integrity monitoring mechanism.

The mechanism can monitor the complete configuration lifecycle:

Input Configuration
        ↓
Input Validation
        ↓
Configuration Transformation
        ↓
Generated Configuration
        ↓
Generated Configuration Validation
        ↓
Service Initialization
        ↓
Runtime Communication

The framework may verify:

* Required fields
* Missing fields
* Empty fields
* Data types
* Value ranges
* Schema version
* Configuration version
* Field mapping
* Source-to-destination consistency
* File integrity
* Transformation result
* Configuration dependencies

Sensitive values such as passwords need not be stored in diagnostic logs.

Instead, RAISE can record metadata such as:

Field: mqtt.username
Expected: NON_EMPTY
Observed: EMPTY
Source: config.json
Destination: mqtt.json
Result: INTEGRITY FAILURE

⸻

10. Configuration Transformation Validation

A key feature of the invention is validation of the relationship between an original configuration and generated runtime configuration.

For example:

config.json
facility.username = PRESENT
facility.password = PRESENT
mqtt.server       = PRESENT
             ↓
Configuration Transformation
             ↓
mqtt.json
username = EMPTY
password = EMPTY
server   = PRESENT

RAISE detects the inconsistency before or during service operation.

It can classify the condition as:

Configuration Transformation Integrity Failure

rather than simply reporting:

MQTT Connection Failure

This allows the framework to investigate the failure at the configuration layer instead of repeatedly restarting the MQTT service.

⸻

11. Dependency-Aware Root-Cause Isolation

RAISE maintains a logical dependency relationship between components.

Example:

Wi-Fi
  ↓
Network
  ↓
Server/Broker Reachability
  ↓
MQTT
  ↓
Application

Another dependency chain may be:

SOM
  ↓
RPCA/RPC
  ↓
Master Board
  ↓
CAN
  ↓
Controller A/B/C

When a downstream component reports a failure, RAISE evaluates upstream dependencies before selecting a recovery action.

For example:

MQTT Failure
     ↓
Check Wi-Fi
     ↓
Wi-Fi Healthy
     ↓
Check Network
     ↓
Network Healthy
     ↓
Check Server/Broker
     ↓
Reachable
     ↓
Check MQTT Configuration
     ↓
mqtt.json Username = EMPTY
     ↓
Check Source Configuration
     ↓
Source Username = PRESENT
     ↓
Configuration Transformation Integrity Failure

This prevents unnecessary recovery actions such as rebooting the entire SOM when the actual issue is a generated configuration inconsistency.

⸻

12. Runtime Configuration and Communication Correlation

RAISE correlates configuration state with runtime communication state.

For example:

MQTT Authentication Failure
          +
Wi-Fi Healthy
          +
Network Healthy
          +
Broker Reachable
          +
mqtt.json Credential Field Invalid
          +
Source Configuration Field Valid

The framework can classify the probable root cause as:

Configuration Processing / Transformation Failure

This correlation capability allows the system to distinguish between:

* Network failure
* Wi-Fi failure
* Server failure
* MQTT service failure
* Authentication failure
* Configuration failure
* Configuration transformation failure

⸻

13. Hierarchical Recovery Mechanism

RAISE uses hierarchical recovery instead of immediately rebooting the complete system.

A representative recovery hierarchy is:

Level 0 – Monitor

Continue monitoring without intervention.

Level 1 – Retry

Retry the failed operation.

Level 2 – Reload / Regenerate Configuration

Reload or regenerate the affected configuration when applicable.

Level 3 – Reconnect Communication

Reconnect Wi-Fi, MQTT, RPCA/RPC or CAN communication.

Level 4 – Restart Affected Service

Restart only the failed software service.

Level 5 – Reset Communication Interface

Reset the affected communication subsystem.

Level 6 – Reset Affected Controller

Reset the affected Master Board or subordinate controller.

Level 7 – Restart Subsystem

Restart the affected subsystem.

Level 8 – Controlled SOM Reboot

Perform a controlled SOM reboot when lower-level recovery actions fail.

The exact recovery hierarchy can be configured for each product.

⸻

14. Recovery Validation

A recovery action is not considered successful merely because the process restarted.

RAISE performs post-recovery validation.

For example:

Restart MQTT
      ↓
Check MQTT Process
      ↓
Check Network
      ↓
Check Broker Reachability
      ↓
Establish MQTT Session
      ↓
Verify Required MQTT Operation
      ↓
Heartbeat / Publish / Subscribe Validation
      ↓
Recovery Successful

If validation fails:

Recovery Failed
      ↓
Execute Next Recovery Level

This prevents false recovery reporting.

⸻

15. MQTT Failure Recovery

Example:

MQTT heartbeat timeout
        ↓
Check Wi-Fi
        ↓
Check Network
        ↓
Check Broker Reachability
        ↓
Check MQTT Configuration
        ↓
Determine Failure Category
        ↓
Reconnect MQTT
        ↓
Validate MQTT

If unsuccessful:

Reconnect
   ↓
Restart MQTT Service
   ↓
Reset Network Interface
   ↓
Regenerate/Reload Configuration if applicable
   ↓
Validate
   ↓
Controlled SOM Recovery if required

The actual sequence is configurable.

⸻

16. Wi-Fi Failure Recovery

When a Wi-Fi failure is detected:

Wi-Fi Failure
      ↓
Check Interface
      ↓
Check Driver/Interface State
      ↓
Retry Connection
      ↓
Reassociate
      ↓
Validate IP Address
      ↓
Validate Network Reachability
      ↓
Validate Server/Broker
      ↓
Reconnect MQTT
      ↓
Validate End-to-End Communication

The system can avoid rebooting the complete SOM if the Wi-Fi subsystem can be recovered independently.

⸻

17. RPCA/RPC and Master Board Failure

The framework monitors communication between the SOM and Master Board.

Example:

RPCA/RPC Timeout
       ↓
Retry Request
       ↓
Check Master Heartbeat
       ↓
Check Communication Interface
       ↓
Perform Diagnostic Request
       ↓
Reset Master Board if permitted
       ↓
Wait for Initialization
       ↓
Validate Master
       ↓
Validate CAN Communication

If the Master Board remains unavailable, RAISE records the failure and escalates according to configured product safety and functional requirements.

⸻

18. CAN and STM Controller Failure

The Master Board may communicate with multiple controllers.

For example:

Master Board
     │
     ├── Controller A → Healthy
     ├── Controller B → No Response
     └── Controller C → Healthy

RAISE can use this information to isolate the likely failure to Controller B or its communication path rather than declaring the entire CAN system failed.

The framework may perform:

1. Retry communication
2. Check node heartbeat
3. Send diagnostic request
4. Check node status
5. Reset affected node if supported
6. Validate node response
7. Mark node as degraded/failed if recovery is unsuccessful
8. Preserve unaffected functionality where permitted

⸻

19. Software Crash Recovery

RAISE can monitor application/service health through:

* Process existence
* Heartbeat
* IPC/RPC response
* Expected state
* Response timeout
* Restart count

Example:

Application Heartbeat Missing
          ↓
Check Process
          ↓
Collect Diagnostic Information
          ↓
Restart Application
          ↓
Validate Application
          ↓
If Failed → Restart Subsystem
          ↓
If Failed → Escalate

⸻

20. Fault History and Diagnostic Information

RAISE maintains a structured fault history.

A fault record may contain:

Fault ID
Component
Fault Category
Timestamp
Observed Symptoms
Probable Root Cause
Recovery Level
Recovery Action
Validation Result
Recovery Duration
Occurrence Count
Current State

Sensitive credentials or secrets are not required to be stored.

The history can be used to identify recurring failures.

⸻

21. Repeated Failure Correlation

RAISE can identify repeated failures over time.

Example:

MQTT Authentication Failure
       ↓
Configuration Field Invalid
       ↓
Recovery
       ↓
System Restored
Later:
MQTT Authentication Failure
       ↓
Same Configuration Field Invalid
       ↓
Recovery
       ↓
System Restored

After repeated occurrences, the framework can classify the condition as a recurring configuration or software issue.

This can allow the product to provide more meaningful diagnostics instead of treating every occurrence as an independent MQTT failure.

⸻

22. Deterministic Operation

The core RAISE recovery mechanism can operate using deterministic rules.

It does not require:

* AI
* Machine learning
* Cloud connectivity
* External servers
* Human intervention

for normal recovery operation.

The system can use predefined dependency relationships, health thresholds and recovery policies.

This makes the framework suitable for embedded systems where predictable behavior and controlled recovery are important.

⸻

23. Configuration-Driven Design

RAISE can use YAML or another structured configuration format to define product-specific policies.

Example:

components:
  mqtt:
    heartbeat_timeout: 30
    retry_count: 3
    recovery:
      - reconnect
      - restart_service
      - reset_network
      - reboot_som
  master_board:
    heartbeat_timeout: 5
    recovery:
      - retry_rpca
      - reset_master
      - validate_can
  can_node:
    response_timeout: 500
    recovery:
      - retry
      - diagnostic_request
      - reset_node

The framework therefore separates:

Generic RAISE Engine

from:

Product-Specific Health and Recovery Policies

This allows the same core framework to be reused across multiple embedded products.

⸻

24. Implementation Technology

A representative implementation may use:

C++

For:

* RAISE core engine
* Health Manager
* Fault Manager
* Dependency Manager
* Recovery Engine
* Configuration Manager
* Validation Engine
* Diagnostic Manager
* Fault History
* Recovery state machine

C

For:

* Low-level hardware interfaces
* CAN communication
* GPIO
* Hardware reset
* Watchdog
* Low-level controller communication

Embedded Linux

For:

* Process/service monitoring
* Runtime state
* File-system operations
* Configuration management
* Logging
* Inter-process communication
* Network monitoring

YAML / Structured Configuration

For:

* Health thresholds
* Heartbeat intervals
* Timeout values
* Retry counts
* Component dependencies
* Recovery policies

⸻

25. Product-Independent Architecture

The proposed invention is not restricted to a particular processor.

For example, the initial implementation may run on an NXP i.MX6-based SOM and may later be adapted to another SOM/platform.

The architecture separates:

RAISE Core
     +
Product Configuration
     +
Hardware Adapter Layer
     +
Communication Adapter Layer

Therefore, the same RAISE concept can be reused across different embedded products.

⸻

26. Traditional Approach vs RAISE

Traditional System	RAISE
Detects individual errors	Correlates multiple symptoms
Watchdog reset	Hierarchical recovery
Generic MQTT failure	Investigates underlying dependency
Manual configuration inspection	Automated configuration integrity checking
Reboot frequently used	Minimum-impact recovery
Limited root-cause information	Probable root-cause isolation
Recovery may not be validated	Post-recovery functional validation
Product-specific monitoring	Reusable framework
Static fault logs	Fault and recovery history
Independent component monitoring	Dependency-aware monitoring

⸻

27. Key Benefits

The invention can provide:

* Reduced product downtime
* Reduced manual troubleshooting
* Faster fault isolation
* Automatic recovery
* Reduced unnecessary full-system reboot
* Improved configuration reliability
* Improved runtime reliability
* Better diagnostic information
* Detection of configuration transformation errors
* Reusable architecture
* Product-independent framework
* Deterministic recovery
* Improved maintainability
* Reduced field-service effort

⸻

28. Difference from Conventional Watchdog

A conventional watchdog generally answers:

“Is the system still running?”

RAISE attempts to answer:

“What failed, what is the probable root cause, what is the least disruptive recovery action, did the recovery actually work, and what should happen next if it did not?”

Therefore:

Watchdog:
Failure
  ↓
Reset
RAISE:
Failure
  ↓
Detect
  ↓
Correlate
  ↓
Diagnose
  ↓
Select Recovery
  ↓
Recover
  ↓
Validate
  ↓
Escalate if Required
  ↓
Record

The watchdog can still be used as one of the recovery mechanisms within RAISE.

⸻

29. Potential Inventive Features

The potentially patentable aspects include the combination of:

1. SOM-centric centralized runtime health supervision of a distributed embedded system.
2. Dependency-aware root-cause isolation across multiple embedded communication layers.
3. Configuration lifecycle integrity monitoring from source configuration through transformed runtime configuration.
4. Automated detection of inconsistencies between source configuration and generated runtime configuration.
5. Correlation of runtime communication failures with configuration transformation failures.
6. Hierarchical minimum-impact recovery based on the identified affected component.
7. Independent post-recovery functional validation.
8. Automatic escalation when recovery validation fails.
9. Historical correlation of recurring failures.
10. Configuration-driven recovery policies allowing a reusable framework to operate across different products.
11. Product-independent RAISE core separated from product-specific hardware and recovery adapters.
12. Deterministic local operation without mandatory AI/ML or cloud dependency.

⸻

30. Proposed Independent Claim Concept

A possible independent claim concept is:

A runtime autonomous health monitoring and self-healing system for a distributed embedded system comprising a Linux-based system-on-module, a master controller and one or more subordinate controllers, wherein the system-on-module executes a runtime health engine configured to monitor health states of software services, communication interfaces, configuration data and subordinate controllers; correlate failure information according to predefined component dependencies; identify a probable root cause; select and execute a hierarchical recovery action associated with the identified component; independently validate functional recovery; and automatically escalate to a higher-level recovery action when validation fails.

A further claim may cover:

Verification of consistency between an input configuration and a generated runtime configuration, identification of a configuration transformation integrity failure based on inconsistency between corresponding source and destination configuration fields, and correlation of the identified integrity failure with a runtime communication failure.

⸻

31. End-to-End Example

A complete example of operation is:

Product Starts
      ↓
RAISE Starts
      ↓
Load Health Policies
      ↓
Monitor SOM
      ↓
Monitor Wi-Fi
      ↓
Monitor Network
      ↓
Monitor MQTT
      ↓
Monitor Configuration
      ↓
Monitor RPCA/RPC
      ↓
Monitor Master Board
      ↓
Monitor CAN
      ↓
Monitor STM Controllers

Assume MQTT authentication fails:

MQTT Failure
      ↓
Check Wi-Fi
      ↓
Wi-Fi Healthy
      ↓
Check Network
      ↓
Network Healthy
      ↓
Check Broker
      ↓
Broker Reachable
      ↓
Check mqtt.json
      ↓
Username = EMPTY
      ↓
Check config.json
      ↓
Username = PRESENT
      ↓
Configuration Transformation Integrity Failure
      ↓
Regenerate/Correct Configuration if Safe
      ↓
Restart MQTT
      ↓
Validate MQTT Authentication
      ↓
Validate MQTT Communication
      ↓
Recovery Successful
      ↓
Record Fault and Recovery

If recovery fails:

Recovery Failed
      ↓
Retry
      ↓
Restart Service
      ↓
Reset Network
      ↓
Revalidate
      ↓
Controlled SOM Recovery if Required
      ↓
Escalate / Record Persistent Failure

⸻

32. One-Line Invention Statement

RAISE is an autonomous embedded runtime framework that detects failures, correlates dependent system symptoms, verifies configuration integrity, isolates probable root causes, performs minimum-impact hierarchical recovery, validates the recovery and escalates automatically when recovery is unsuccessful.

⸻

33. Initial Proof-of-Concept Scope

The initial proof of concept can focus on the embedded product side only.

Phase 1 – SOM

Implement:

* RAISE core
* Health monitoring
* Process monitoring
* Wi-Fi monitoring
* MQTT monitoring
* Configuration validation
* Fault history

Phase 2 – Configuration Integrity

Implement:

config.json
      ↓
Transformation
      ↓
mqtt.json
      ↓
Integrity Validation

Detect:

* Missing fields
* Empty fields
* Invalid fields
* Source/destination mismatch
* Transformation failure

Phase 3 – Communication Recovery

Implement:

* MQTT reconnect
* MQTT service restart
* Wi-Fi recovery
* Network recovery

Phase 4 – RPCA/Master

Implement:

* RPCA heartbeat
* Timeout detection
* Master recovery
* Post-recovery validation

Phase 5 – CAN Controllers

Implement:

* CAN node heartbeat
* Node timeout detection
* Node isolation
* Diagnostic request
* Node recovery
* Recovery validation

Phase 6 – Fault Correlation

Implement:

* Fault history
* Recurring fault detection
* Root-cause correlation
* Recovery statistics

⸻

34. Expected Proof-of-Concept Result

The POC should demonstrate that the system can intentionally introduce controlled failures such as:

* MQTT disconnection
* Wi-Fi disconnection
* Invalid MQTT configuration
* Missing configuration field
* Empty generated configuration field
* Application crash
* RPCA timeout
* Master Board communication failure
* CAN node timeout

and automatically perform:

Detection
   ↓
Diagnosis
   ↓
Recovery
   ↓
Validation
   ↓
Escalation if Required

The POC should also demonstrate that RAISE can distinguish between a communication symptom and a configuration/root-cause failure.

⸻

35. Scope Boundary for Initial Invention

The initial implementation and patent disclosure are focused on embedded/product-side autonomous health monitoring and self-healing.

The system may later be extended to monitor and recover cloud/server-side infrastructure, but such functionality is outside the scope of the initial implementation and POC.

The first implementation therefore concentrates on:

SOM → Wi-Fi/Network → MQTT → Configuration → RPCA/RPC → Master Board → CAN → STM Controllers

⸻

36. Final Summary

RAISE provides a reusable architecture for autonomous reliability of distributed embedded products.

Its main concept is not simply restarting a failed component.

The framework establishes a complete autonomous lifecycle:

Detect → Correlate → Diagnose → Recover → Validate → Escalate → Learn from Fault History

with particular emphasis on:

runtime health + dependency-aware diagnosis + configuration integrity + hierarchical recovery + independent validation.

This combination is intended to reduce field failures, minimize unnecessary system reboots, improve diagnostics and provide autonomous recovery for distributed embedded systems.
