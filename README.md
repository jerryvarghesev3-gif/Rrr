PATENT INVENTION DISCLOSURE

1. TITLE OF THE INVENTION

Autonomous Runtime Health Monitoring, Configuration Integrity Verification, Root-Cause Isolation and Hierarchical Self-Healing Framework for Distributed Embedded Systems

Short Name

RAISE — Runtime Autonomous Intelligent Self-Healing Engine

⸻

2. TECHNICAL FIELD

The invention relates to embedded systems, distributed embedded architectures, runtime health monitoring, configuration integrity, fault detection, root-cause isolation, communication management, diagnostics, and autonomous system recovery.

More particularly, the invention relates to a system-on-module (SOM)-based supervisory framework configured to monitor software services, communication interfaces, configuration data, controllers, and distributed embedded boards; identify runtime failures and configuration-related failures; determine a probable root cause using deterministic dependency-aware analysis; execute a hierarchical recovery operation; and independently validate whether the affected functionality has been restored.

The framework is particularly applicable to embedded products comprising a SOM, network/Wi-Fi connectivity, MQTT or equivalent communication protocols, RPC/RPCA communication, a master controller, CAN communication, and multiple subordinate microcontroller-based boards.

⸻

3. BACKGROUND / PROBLEM STATEMENT

Modern embedded products commonly consist of multiple interconnected hardware and software components.

A typical system may include:

* A System-on-Module (SOM)
* Embedded Linux
* Wi-Fi/network interfaces
* MQTT communication
* Server/cloud connectivity
* RPC/RPCA communication
* Master Board
* CAN communication
* Multiple STM-based subordinate boards
* Application services
* Configuration files
* Hardware I/O interfaces
* Local file systems

Failures may occur at any layer of the system.

Examples include:

* Wi-Fi disconnection
* Network connectivity failure
* MQTT disconnection
* MQTT authentication failure
* Server disconnection
* RPC/RPCA communication timeout
* Master Board failure
* CAN communication failure
* Individual CAN-node failure
* Software process crash
* Software service termination
* Unresponsive task
* Configuration file corruption
* Missing configuration field
* Empty configuration value
* Incorrect configuration transformation
* Configuration generation failure
* Partial file update
* Incorrect mapping between configuration files
* Unexpected reboot during configuration update
* Race conditions during configuration generation
* Storage/file-system-related configuration corruption
* Repeated transient failures

In conventional embedded architectures, individual software modules often implement their own recovery mechanisms.

For example:

MQTT Module
    → MQTT reconnect
Wi-Fi Module
    → Wi-Fi reconnect
CAN Module
    → CAN retry
Application
    → Restart process
Watchdog
    → Reboot device

Such recovery mechanisms are generally isolated and may not understand the dependency relationship between different components.

Consequently, a downstream failure may be incorrectly treated as the primary failure.

For example:

Wi-Fi Failure
     ↓
MQTT Failure
     ↓
Application Failure

The application may only detect the MQTT failure even though the underlying cause is the Wi-Fi failure.

A similar problem can occur during configuration generation.

For example:

config.json
     ↓
Configuration processing
     ↓
mqtt.json
     ↓
MQTT service
     ↓
Server

A configuration file may contain required credentials and parameters, but during a transformation or generation process, one or more fields may become empty, missing, invalid, or incorrectly mapped.

For example:

config.json
facility.username = "configured value"
facility.password = "configured value"

while the generated file contains:

mqtt.json
username = ""
password = ""

The MQTT service subsequently fails authentication.

In such a case, the final observable symptom is an MQTT failure, while the actual cause may have occurred earlier during configuration processing.

The exact reason may be unknown and could involve multiple possible failure mechanisms.

There is therefore a need for a unified framework capable of monitoring the complete runtime chain, validating configuration integrity at different stages, correlating failures with component dependencies, identifying a probable root cause, automatically applying an appropriate recovery action, and validating the result.

⸻

4. SUMMARY OF THE INVENTION

The proposed invention provides an autonomous runtime health, configuration-integrity, fault-diagnosis, and self-healing framework implemented primarily on a System-on-Module (SOM).

The SOM acts as a supervisory health-management node for a distributed embedded system.

The distributed system may comprise:

* SOM
* Wi-Fi/network subsystem
* MQTT or equivalent communication service
* Server/cloud communication
* Application software
* RPC/RPCA communication
* Master Board
* CAN bus
* Multiple STM-based subordinate boards
* Configuration files
* Embedded file system and I/O interfaces

The framework continuously monitors the operational health of the components and maintains a dependency model describing relationships among the components.

In addition to runtime communication and process health, the framework monitors the integrity and consistency of configuration data across its lifecycle.

For example:

Configuration Input
        ↓
Configuration Validation
        ↓
Configuration Transformation
        ↓
Generated Configuration
        ↓
Service Initialization
        ↓
Authentication
        ↓
Communication
        ↓
Runtime Operation

When a fault is detected, the framework determines whether the observed failure is a primary failure or a downstream symptom.

The framework can inspect upstream dependencies, configuration state, communication state, process state, heartbeat information, timeout conditions, and historical fault information to identify a probable root cause.

A deterministic recovery policy then selects the least disruptive applicable recovery operation.

Recovery may include:

* Retrying an operation
* Regenerating configuration
* Reconnecting a communication session
* Restarting a service
* Resetting a communication interface
* Reinitializing a hardware interface
* Resetting an affected controller
* Resetting an affected CAN node
* Restarting a subsystem
* Controlled SOM reboot

Following recovery, an independent validation mechanism verifies whether the affected functionality has actually been restored.

If validation fails, the framework automatically escalates to another recovery level.

The framework can operate without artificial intelligence or machine learning and can instead use deterministic rules, state machines, dependency relationships, configuration schemas, health thresholds, fault signatures, and predefined recovery policies.

⸻

5. SYSTEM ARCHITECTURE

The proposed architecture is:

                     SERVER / CLOUD
                          |
                     MQTT / NETWORK
                          |
                +---------v---------+
                |       SOM         |
                |                   |
                |      RAISE        |
                |                   |
                | +---------------+ |
                | | Runtime Health | |
                | | Monitor        | |
                | +-------+-------+ |
                |         |         |
                | +-------v-------+ |
                | | Configuration | |
                | | Integrity     | |
                | +-------+-------+ |
                |         |         |
                | +-------v-------+ |
                | | Fault         | |
                | | Detection     | |
                | +-------+-------+ |
                |         |         |
                | +-------v-------+ |
                | | Dependency &  | |
                | | Root Cause    | |
                | +-------+-------+ |
                |         |         |
                | +-------v-------+ |
                | | Recovery      | |
                | | Engine        | |
                | +-------+-------+ |
                |         |         |
                | +-------v-------+ |
                | | Recovery      | |
                | | Validator     | |
                | +---------------+ |
                +---------+---------+
                          |
                       RPC/RPCA
                          |
                +---------v---------+
                |   MASTER BOARD    |
                +---------+---------+
                          |
                         CAN
              +-----------+-----------+
              |           |           |
            STM-A       STM-B       STM-C

⸻

6. MAJOR FUNCTIONAL COMPONENTS

6.1 Runtime Health Monitor

The Runtime Health Monitor continuously monitors:

* SOM health
* Wi-Fi state
* Network state
* MQTT state
* Server connectivity
* Application process state
* RPC/RPCA communication
* Master Board heartbeat
* CAN bus health
* Individual CAN-node health
* STM-board status
* Hardware I/O status
* Watchdog status
* Runtime resource indicators
* Configuration status

Health states may include:

HEALTHY
DEGRADED
FAILED
RECOVERING
UNKNOWN

⸻

7. CONFIGURATION INTEGRITY AND PROVENANCE MANAGER

A major aspect of the invention is monitoring configuration data throughout its lifecycle.

For example:

config.json
      ↓
Input Validation
      ↓
Configuration Transformation
      ↓
mqtt.json
      ↓
MQTT Initialization
      ↓
Authentication
      ↓
MQTT Connection

The framework can validate:

* Required fields
* Missing fields
* Empty values
* Data types
* Configuration schema
* Field mappings
* Configuration versions
* Configuration dependencies
* Transformation results
* File integrity
* Expected configuration relationships

Sensitive credentials are not required to be stored in diagnostic logs.

Instead, the framework can record metadata such as:

Field:
mqtt.username
Expected:
NON_EMPTY
Actual State:
EMPTY
Source:
config.json
Destination:
mqtt.json
Transformation:
FAILED / INCONSISTENT

Actual passwords or secret values need not be recorded.

⸻

8. CONFIGURATION TRANSFORMATION VALIDATION

The framework may maintain a configuration mapping relationship.

For example:

config.json
facility.username
       |
       v
mqtt.json
username

and:

config.json
facility.password
       |
       v
mqtt.json
password

The framework verifies that required source values are correctly represented in the generated destination configuration.

Example:

Source:
facility.username = PRESENT
facility.password = PRESENT
Generated:
mqtt.username = EMPTY
mqtt.password = EMPTY

The framework detects:

CONFIGURATION TRANSFORMATION INTEGRITY FAILURE

This can occur before MQTT communication is attempted.

⸻

9. RUNTIME FAULT DETECTION

The Fault Detection Engine identifies abnormal runtime conditions using deterministic mechanisms.

Examples include:

* Heartbeat timeout
* Communication timeout
* Connection loss
* Process termination
* Service failure
* MQTT authentication failure
* MQTT session failure
* Wi-Fi failure
* RPCA timeout
* CAN-node timeout
* Configuration inconsistency
* Repeated failure
* Unexpected state transition

⸻

10. DEPENDENCY-AWARE ROOT-CAUSE ISOLATION

The framework maintains a dependency relationship between system components.

Example:

Server
↓
Wi-Fi
↓
Network
↓
MQTT
↓
Application
↓
RPCA
↓
Master Board
↓
CAN
↓
STM Nodes

When a downstream component reports a failure, the framework checks the health of its upstream dependencies.

Example:

MQTT FAILED

Wi-Fi       → HEALTHY
Network     → HEALTHY
Server      → REACHABLE
MQTT        → FAILED

Probable root cause:

MQTT session/service/configuration failure

Another example:

MQTT FAILED

Wi-Fi       → FAILED
Network     → FAILED
Server      → UNKNOWN
MQTT        → FAILED

Probable root cause:

Wi-Fi/network failure

Therefore, the framework avoids treating every downstream symptom as an independent failure.

⸻

11. CONFIGURATION-RELATED ROOT-CAUSE ANALYSIS

The framework can connect a runtime failure with an earlier configuration event.

Example:

MQTT Authentication Failure
          ↓
Check MQTT Configuration
          ↓
mqtt.username = EMPTY
          ↓
Check Source Configuration
          ↓
config.json username = PRESENT
          ↓
Check Transformation
          ↓
Value lost during transformation

The framework can therefore report:

Probable Root Cause:
Configuration transformation inconsistency
Observed Symptom:
MQTT authentication failure

This provides substantially more diagnostic information than simply reporting:

MQTT disconnected

⸻

12. HIERARCHICAL RECOVERY ENGINE

The framework uses multiple recovery levels.

Example:

LEVEL 0
Continue monitoring
LEVEL 1
Retry operation
LEVEL 2
Regenerate/reload configuration where applicable
LEVEL 3
Reconnect communication session
LEVEL 4
Restart affected service
LEVEL 5
Reset communication interface
LEVEL 6
Reset affected controller/component
LEVEL 7
Restart subsystem
LEVEL 8
Controlled SOM reboot

The exact recovery actions and levels may be configured according to product requirements.

The framework attempts the lowest-impact applicable recovery operation before escalating.

⸻

13. RECOVERY VALIDATION

A recovery operation is not automatically considered successful after the recovery command completes.

The framework performs independent validation.

Example:

MQTT Failure
     ↓
Reconnect MQTT
     ↓
Connection established?
     ↓
Publish test
     ↓
Expected response?
     ↓
Heartbeat restored?
     ↓
YES
     ↓
MQTT HEALTHY

If validation fails:

Recovery Level 2 FAILED
        ↓
Execute next recovery level
        ↓
Validate again

⸻

14. EXAMPLE — CONFIGURATION FAILURE RESULTING IN MQTT FAILURE

Consider 100 deployed products.

A subset of products may experience MQTT disconnection or authentication failure.

Investigation identifies:

mqtt.json
username = EMPTY
password = EMPTY

However, the original cause is unknown.

The framework can investigate the configuration lifecycle:

config.json
     ↓
Input Validation
     ↓
Transformation
     ↓
mqtt.json Validation
     ↓
MQTT Initialization
     ↓
Authentication
     ↓
Connection

Possible causes can include:

* Missing input field
* Empty input value
* Incorrect field mapping
* Transformation logic failure
* Partial file write
* Interrupted configuration update
* Process crash during generation
* Race condition
* Incorrect default value
* Configuration version mismatch
* File-system failure
* Unexpected reboot
* Software defect
* Other unknown condition

The framework can identify which known validation stage first violated the expected configuration state.

If the exact root cause cannot be determined, the framework records the failure as an unknown or unclassified cause instead of making an unsupported assumption.

⸻

15. CONFIGURATION FINGERPRINT / VERSION TRACKING

The framework may associate configuration states with:

* Configuration version
* Schema version
* Product version
* Firmware version
* Transformation version
* Timestamp
* Configuration fingerprint/checksum
* Generation status

Example:

Configuration ID:
CFG-XXXX
Firmware:
FW-X.X
Schema:
SCHEMA-X.X
Transformation:
TRANSFORM-X.X
Integrity:
VALID

This allows the framework to determine whether an unexpected configuration state occurred during a specific transformation or software version.

⸻

16. FAULT HISTORY

The framework may maintain fault history containing:

* Fault identifier
* Component
* Fault category
* Timestamp
* Detected condition
* Probable root cause
* Recovery operation
* Recovery result
* Recovery duration
* Number of occurrences
* Escalation level

Example:

FAULT:
MQTT_AUTH_FAILURE
OBSERVATION:
MQTT authentication unsuccessful
CONFIGURATION STATE:
mqtt.username = EMPTY
SOURCE CONFIGURATION:
username = PRESENT
PROBABLE ROOT CAUSE:
Configuration transformation inconsistency
RECOVERY:
Configuration regeneration
VALIDATION:
MQTT authentication successful
FINAL STATE:
HEALTHY

Sensitive credential values are not required to be stored.

⸻

17. REPEATED FAILURE CORRELATION

The framework can detect whether a failure is transient or persistent.

Example:

MQTT failure
    ↓
Recovered
    ↓
MQTT failure
    ↓
Recovered
    ↓
MQTT failure
    ↓
Repeated failure threshold reached

The framework can classify the condition as:

PERSISTENT / RECURRING FAULT

and escalate recovery or generate a diagnostic condition.

⸻

18. CAN NODE FAILURE EXAMPLE

Assume:

Master Board = HEALTHY
CAN Bus      = HEALTHY
STM-A = HEALTHY
STM-B = FAILED
STM-C = HEALTHY

Because other CAN nodes remain operational, the framework can isolate the probable failure to STM-B or its communication path.

The framework can execute:

STM-B timeout
      ↓
Retry
      ↓
Diagnostic request
      ↓
Node recovery
      ↓
Validate STM-B

If STM-B remains unavailable:

STM-B = DEGRADED / FAILED

The remaining system may continue operating only where allowed by the product’s functional and safety requirements.

⸻

19. SOFTWARE CRASH EXAMPLE

Suppose an MQTT-related service crashes.

The framework detects:

Process heartbeat missing
        ↓
Process state check
        ↓
Process terminated

Recovery:

Restart service
        ↓
Validate process
        ↓
Validate MQTT
        ↓
Validate server communication

If unsuccessful:

Restart communication subsystem
        ↓
Validate
        ↓
Escalate if required

⸻

20. IMPLEMENTATION TECHNOLOGY

The proposed framework can be implemented using a combination of:

C++

For:

* Runtime Health Manager
* Fault Manager
* Dependency Manager
* Recovery Engine
* State machine
* Configuration Manager
* Validation Engine
* Diagnostic Manager
* Fault-history management

C

For:

* Low-level hardware interfaces
* CAN interface
* GPIO
* STM communication
* Watchdog
* Hardware reset
* Low-level I/O
* Embedded interfaces

YAML

For configurable:

* Health thresholds
* Heartbeat intervals
* Timeout values
* Retry counts
* Recovery levels
* Component dependencies
* Configuration validation rules
* Product-specific recovery policies

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

Embedded Linux File System / I/O

The framework may use the embedded file system for:

* Runtime health state
* Diagnostic state
* Configuration state
* Fault history
* Recovery history
* Status information
* Runtime communication with other processes

The exact file-system structure is implementation-specific.

⸻

21. GENERIC / PRODUCT-INDEPENDENT DESIGN

The framework is designed to separate:

CORE ENGINE

from:

PRODUCT-SPECIFIC CONFIGURATION

Therefore:

Product A
    ↓
Product A YAML
    ↓
RAISE Core
Product B
    ↓
Product B YAML
    ↓
RAISE Core

This allows the same core framework to potentially support different SOM platforms, Master Boards, CAN-node configurations, communication stacks, and product variants.

⸻

22. TRADITIONAL ARCHITECTURE VS PROPOSED FRAMEWORK

Capability	Traditional Architecture	Proposed RAISE Framework
Runtime monitoring	Module-specific	Unified health model
MQTT failure	Reconnect/retry	Diagnose → recover → validate
Wi-Fi failure	Independent recovery	Dependency-aware diagnosis
Server failure	Retry connection	Server/network/MQTT correlation
Configuration failure	Usually discovered later	Validate during lifecycle
Missing configuration field	May reach runtime	Detect before dependent service
Empty MQTT credential	MQTT failure	Trace to configuration transformation
RPCA failure	Retry/reset	Dependency-aware diagnosis
CAN failure	Bus/node retry	Node-level isolation
Software crash	Restart/reboot	Progressive recovery
Root-cause analysis	Limited	Dependency + configuration correlation
Recovery strategy	Usually fixed	Hierarchical
Recovery validation	Limited	Explicit validation
Repeated failures	Basic logging	Fault correlation/escalation
Configuration	Code-heavy	YAML-driven
AI dependency	Not required	Not required
Cloud dependency	Not required	Not required
Full reboot	Often used as recovery	Prefer last-resort recovery
Reusability	Product-specific	Generic framework

⸻

23. KEY BENEFITS

23.1 Improved System Availability

Localized failures can potentially be recovered without restarting the complete product.

23.2 Reduced Unnecessary Reboots

The framework attempts lower-impact recovery operations before full-system reboot.

23.3 Faster Root-Cause Identification

The framework correlates runtime failures with dependencies and configuration state.

23.4 Configuration Error Detection

Invalid, missing, empty, or inconsistent configuration values can be detected before they result in downstream service failures.

23.5 Better Field Diagnostics

The system can provide information about where a failure originated rather than only reporting the final symptom.

23.6 Automated Recovery

Predefined failures can be automatically recovered without requiring manual service intervention.

23.7 Reusable Architecture

The framework can be reused across different products through configuration and hardware adapters.

23.8 Deterministic Operation

The framework can operate using predefined rules, state machines, dependency relationships, and recovery policies without requiring AI or machine learning.

23.9 Improved Fault Traceability

Configuration and runtime events can be correlated to identify the stage at which an unexpected condition first occurred.

⸻

24. DIFFERENCE FROM A CONVENTIONAL WATCHDOG

A conventional watchdog primarily detects that a system or task has become unresponsive and performs a reset.

The proposed framework performs:

DETECT
   ↓
CORRELATE
   ↓
DIAGNOSE
   ↓
SELECT RECOVERY
   ↓
EXECUTE
   ↓
VALIDATE
   ↓
ESCALATE IF REQUIRED

A watchdog may still be used as one of the recovery mechanisms within the proposed framework.

⸻

25. POTENTIAL INVENTIVE FEATURES

The invention may particularly emphasize the combination of the following:

1. SOM-based centralized runtime health supervision of a distributed embedded architecture.
2. Dependency-aware fault isolation across network, MQTT, RPC/RPCA, Master Board, CAN, and subordinate embedded nodes.
3. Configuration lifecycle integrity monitoring from source configuration through transformation and generated configuration.
4. Correlation of runtime communication failures with configuration inconsistencies.
5. Hierarchical minimum-impact recovery based on fault type and system dependency.
6. Independent functional validation after each recovery action.
7. Automatic escalation when validation fails.
8. Historical correlation of repeated faults.
9. Configuration-driven recovery policies using YAML or equivalent configuration mechanisms.
10. Deterministic autonomous operation without requiring AI, machine learning, or cloud-based decision making.
11. Localized recovery of individual distributed components while preserving operation of unaffected components where permitted.
12. Separation of a reusable self-healing engine from product-specific configuration and hardware adapters.

⸻

26. PROPOSED INDEPENDENT CLAIM CONCEPT

The following is a technical concept for patent counsel to refine:

A system for autonomous runtime fault management of a distributed embedded system, comprising:

a system-on-module configured to communicate with a master controller and a plurality of subordinate embedded controllers;

a plurality of communication interfaces including a network interface and a controller-area-network interface;

a configuration integrity module configured to validate configuration information at one or more stages between an input configuration and a generated configuration used by a software service;

a runtime health monitoring module configured to obtain health indicators associated with software services, communication interfaces, the master controller, and the plurality of subordinate embedded controllers;

a dependency model representing relationships among the software services, communication interfaces, configuration information, master controller, and subordinate embedded controllers;

a fault detection and classification module configured to detect an abnormal operating condition and determine a probable root cause based on at least one of the health indicators, configuration integrity state, dependency relationships, timeout conditions, heartbeat information, and historical fault information;

a recovery decision module configured to select a recovery operation from a plurality of hierarchical recovery operations based on the probable root cause;

a recovery execution module configured to execute the selected recovery operation; and

a recovery validation module configured to determine whether functionality associated with the detected abnormal operating condition has been restored following execution of the recovery operation, wherein failure of the recovery validation causes selection of a subsequent recovery operation having a higher recovery level.

⸻

27. EXAMPLE END-TO-END OPERATION

              CONFIGURATION RECEIVED
                       |
                       v
               Validate Config
                       |
                       v
              Transform Config
                       |
                       v
             Validate Generated
                 Configuration
                       |
                       v
               Start Services
                       |
                       v
             Runtime Monitoring
                       |
                       v
                Fault Detected
                       |
                       v
             Check Dependencies
                       |
                       v
             Check Configuration
                       |
                       v
             Identify Root Cause
                       |
                       v
             Select Recovery
                       |
                       v
             Execute Recovery
                       |
                       v
             Validate Recovery
                    /     \
                  PASS     FAIL
                   |         |
                   v         v
                HEALTHY   ESCALATE
                              |
                              v
                       Next Recovery
                              |
                              v
                           Validate

⸻

28. ONE-LINE DESCRIPTION OF THE INVENTION

“A SOM-based autonomous framework that continuously monitors runtime behavior and configuration integrity across a distributed embedded system, correlates failures through component dependencies to identify probable root causes, performs progressively escalating minimum-impact recovery actions, and independently validates recovery before escalation.”

⸻

29. EXPECTED OUTCOME

The proposed framework is intended to transform the conventional embedded architecture from a primarily failure-detection-and-reset model into an autonomous runtime health-management and self-healing model.

Instead of only detecting:

MQTT DISCONNECTED

the framework attempts to determine:

MQTT DISCONNECTED
       ↓
Wi-Fi HEALTHY
Network HEALTHY
Server REACHABLE
       ↓
MQTT AUTHENTICATION FAILURE
       ↓
mqtt.json credential field EMPTY
       ↓
Source config field PRESENT
       ↓
Configuration transformation inconsistency
       ↓
Regenerate configuration
       ↓
Validate MQTT authentication
       ↓
MQTT HEALTHY

Where the exact root cause cannot be determined, the framework records the known observations and performs the applicable recovery and escalation process without making an unsupported root-cause assumption.

This provides a generalized architecture for autonomous fault detection, diagnosis, configuration-integrity verification, recovery, validation, and continued operation of distributed embedded products.
