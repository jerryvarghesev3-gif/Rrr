“Autonomous Runtime Fault Detection, Root-Cause Isolation and Hierarchical Self-Healing Framework for Distributed Embedded Systems”


Core invention

A SOM acts as an autonomous health-management controller for a distributed embedded system consisting of:

* SOM / Linux gateway
* Wi-Fi/network stack
* MQTT/application services
* RPC/RPCA communication
* Master Board
* Multiple CAN-connected slave boards
* Runtime health monitoring
* Deterministic fault diagnosis
* Hierarchical recovery
* Recovery validation
* Escalation/rollback

The key is that the SOM doesn’t simply detect that something failed.


It determines:

Which layer failed → why it probably failed → what is the minimum recovery action → whether recovery succeeded → what escalation is required if it didn’t.

And this happens locally, without AI or cloud dependency.


1. Your system architecture

I would structure your patent around this:

                         CLOUD / SERVER
                              │
                         MQTT / Network
                              │
                    ┌─────────▼─────────┐
                    │       SOM         │
                    │                   │
                    │ Runtime Health    │
                    │ Management Engine │
                    │                   │
                    │ ┌───────────────┐ │
                    │ │ Health        │ │
                    │ │ Monitor       │ │
                    │ └───────┬───────┘ │
                    │         ↓         │
                    │ ┌───────────────┐ │
                    │ │ Fault         │ │
                    │ │ Detection     │ │
                    │ └───────┬───────┘ │
                    │         ↓         │
                    │ ┌───────────────┐ │
                    │ │ Root Cause    │ │
                    │ │ Isolation     │ │
                    │ └───────┬───────┘ │
                    │         ↓         │
                    │ ┌───────────────┐ │
                    │ │ Recovery      │ │
                    │ │ Decision      │ │
                    │ └───────┬───────┘ │
                    │         ↓         │
                    │ ┌───────────────┐ │
                    │ │ Recovery      │ │
                    │ │ Validation    │ │
                    │ └───────────────┘ │
                    └─────────┬─────────┘
                              │
                           RPCA/RPC
                              │
                    ┌─────────▼─────────┐
                    │   MASTER BOARD    │
                    └────┬────┬────┬────┘
                         │    │    │
                        CAN  CAN  CAN
                         │    │    │
                       ┌─▼┐  ┌▼─┐  ┌▼─┐
                       │ A│  │ B│  │ C│
                       └──┘  └──┘  └──┘


                       
2. The important invention: a Health Dependency Graph

This is one of the areas I’d emphasize.

The SOM doesn’t monitor everything independently.

It understands the dependency relationship.

For example:

Server
  │
  ▼
Wi-Fi
  │
  ▼
IP Network
  │
  ▼
MQTT
  │
  ▼
Application
  │
  ▼
RPCA
  │
  ▼
Master Board
  │
  ▼
CAN
 ┌┼───────┐
 A        B        C



 Suppose MQTT stops receiving messages.

Your system doesn’t immediately conclude:

MQTT is broken.

It checks its dependencies.


MQTT failure
     │
     ├── Wi-Fi?       YES
     ├── IP?          YES
     ├── Server?      YES
     ├── TCP?         YES
     └── MQTT?        NO



Therefore:

Root cause = MQTT service/session failure

Then the system performs the minimum recovery action.

⸻

3. Example — MQTT failure

Your patent workflow could specify:


MQTT heartbeat timeout
        ↓
Check Wi-Fi health
        ↓
Check network reachability
        ↓
Check server reachability
        ↓
Check MQTT socket/session
        ↓
Classify failure
        ↓
Attempt MQTT reconnect
        ↓
Validate MQTT publish/subscribe

If successful:

HEALTHY

If unsuccessful:

Recovery Level 2
      ↓
Restart MQTT service
      ↓
Validate


Still unsuccessful:

Recovery Level 3
      ↓
Restart network interface
      ↓
Validate

Still unsuccessful:

Recovery Level 4
      ↓
Restart SOM communication subsystem

Finally:

Recovery Level 5
      ↓
Controlled SOM reboot

4. Example — Master Board failure

This is where your architecture becomes much more interesting.

Suppose:

Wi-Fi       HEALTHY
MQTT        HEALTHY
SOM         HEALTHY
RPCA        FAILED
Master      NO RESPONSE


The SOM doesn’t reboot itself immediately.

It performs:

RPCA heartbeat timeout
       ↓
RPCA retry
       ↓
Master Board diagnostic request
       ↓
No response
       ↓
Classify Master Board communication failure
       ↓
Attempt RPCA recovery
       ↓
Validate Master Board heartbeat

If your hardware supports a reset mechanism:

Master Board reset
       ↓
Wait for boot
       ↓
Check Master heartbeat
       ↓
Check CAN bus
       ↓
Check A/B/C nodes

5. Example — CAN Board B failure

This is potentially one of the strongest examples for your patent.

Imagine:

Master Board       HEALTHY

CAN Bus            HEALTHY

Board A             HEALTHY
Board B             FAILED
Board C             HEALTHY

The SOM can determine:

CAN communication failure
        ↓
Identify Node B
        ↓
Check Node B heartbeat
        ↓
Check CAN response
        ↓
Check Master Board
        ↓
Check CAN bus

Because A and C are responding, it has evidence that:

The CAN bus itself is probably healthy and the failure is localized to Board B.

Then your recovery policy could be:

Node B timeout
      ↓
Retry communication
      ↓
Node B diagnostic request
      ↓
Node B recovery command
      ↓
Node B reset
      ↓
Validate Node B

If B remains unavailable:

Mark B DEGRADED/FAILED
      ↓
Protect remaining system
      ↓
Continue A + C operation
      ↓
Report diagnostic information

That is much more sophisticated than a conventional watchdog.

⸻

6. Your hierarchical recovery mechanism

I would make this central to the patent.


Recovery levels

Level

Action

0

Continue monitoring

1

Retry failed transaction

2

Reconnect component

3

Restart service/process

4

Reset communication interface

5

Reset affected board/module

6

Restart SOM subsystem

7

Controlled SOM reboot

8

System-level recovery



The system always attempts the least disruptive valid recovery first.

This gives you:

Minimal-impact hierarchical recovery

rather than simply:

“Failure → reboot.”


7. The recovery validator

I would make this a major element of your invention.

After every recovery:
             FAILURE
                ↓
        Recovery Action
                ↓
       Recovery Completed
                ↓
          VALIDATION
          /        \
       PASS        FAIL
        ↓            ↓
    HEALTHY      Next recovery
                     level

                     
For example:

MQTT disconnected
       ↓
Reconnect
       ↓
Connected?
       ↓
YES
       ↓
Publish test
       ↓
Receive expected response?
       ↓
YES
       ↓
MQTT = HEALTHY



So merely establishing a connection isn’t considered recovery.

The system proves that the service actually recovered.

That’s an important distinction.

⸻

8. No AI is actually a benefit here

I would explicitly position this as:

Deterministic / rule-based autonomous recovery

The system uses:

* Heartbeats
* Timeouts
* Counters
* State machines
* Dependency relationships
* Fault signatures
* Recovery policies
* Escalation rules
* Validation criteria

For example:

IF

Wi-Fi = HEALTHY
AND
Server = REACHABLE
AND
MQTT heartbeat = TIMEOUT

THEN

Fault = MQTT_SESSION_FAILURE

Recovery = MQTT_RECONNECT

No neural network is required.

That can be valuable for embedded/regulated environments because the behavior is predictable and testable.

9. One feature I’d add: fault signature history

This could make your idea stronger.

The SOM stores:

Fault ID
Timestamp
Component
Symptoms
Detected root cause
Recovery attempted
Recovery result
Recovery duration
Number of occurrences

Example:
FAULT #1042

Component:
MQTT

Symptoms:
Heartbeat timeout

Root Cause:
MQTT session failure

Recovery:
Reconnect

Result:
SUCCESS

Recovery Time:
3.2 seconds

Then you can identify recurring failures.

For example:

MQTT failure
      ↓
Recovered
      ↓
10 minutes later
      ↓
MQTT failure
      ↓
Recovered
      ↓
Again
      ↓
Repeated failure threshold

Then:

Transient failure
        ↓
becomes
Persistent failure

and the recovery strategy escalates.

10. A particularly interesting extension

I’d call this:

Recovery Confidence / Fault Correlation

Suppose the SOM sees:

MQTT disconnected
Wi-Fi still connected
CAN healthy
RPCA healthy
Server reachable


It has strong evidence that MQTT itself is the problem.

But suppose:

Wi-Fi disconnected
MQTT disconnected
Server unreachable

Now MQTT isn’t necessarily the root cause.
The SOM understands:
Wi-Fi
 ↓
MQTT
 ↓
Application

Therefore it can avoid incorrectly recovering the downstream components.

That gives you:

Dependency-aware root-cause isolation

This is much more interesting for a patent than merely “monitoring MQTT.”

⸻

11. What I would NOT claim

Don’t claim:

“The invention detects any vulnerability.”

That’s too broad and problematic.

Instead say:

runtime faults, communication failures, service failures, component failures and abnormal operating conditions.

And distinguish fault from security vulnerability.

Your original word “vulnerability” is potentially misleading here.


Proposed patent abstract

Here is the direction I would use for your invention disclosure:




Title: Autonomous Runtime Fault Detection, Root-Cause Isolation and Hierarchical Self-Healing Framework for Distributed Embedded Systems

Abstract

A computer-implemented and embedded-system framework is provided for autonomous runtime fault detection, root-cause isolation, recovery, and validation in a distributed embedded system. The system includes a system-on-module (SOM) configured as a communication and supervisory node coupled to a master controller and one or more subordinate embedded boards through one or more communication interfaces including RPC/RPCA and CAN. The SOM continuously monitors health indicators associated with communication interfaces, network connectivity, software services, application processes, master-controller communication, CAN nodes, and other system components.

Upon detecting an abnormal operating condition, the framework evaluates a dependency relationship among monitored components to distinguish a primary fault from downstream symptoms. A deterministic fault classification mechanism identifies a probable failed component or communication layer based on predefined health indicators, heartbeat status, timeout conditions, response patterns, and fault signatures. A hierarchical recovery engine then selects a least-disruptive recovery action associated with the identified fault.

The recovery actions may include retrying an operation, reconnecting a communication session, restarting a software service, resetting a communication interface, resetting an affected controller or subordinate board, restarting a subsystem, or performing a controlled system reboot. Following each recovery action, a recovery-validation mechanism independently verifies restoration of the affected functionality using predefined health and functional validation criteria. If validation fails, the framework automatically escalates to a subsequent recovery level.

The framework thereby provides deterministic, autonomous and dependency-aware runtime self-healing of distributed embedded systems without requiring artificial intelligence, cloud connectivity, or manual operator intervention for identified fault conditions.






This is not yet a final legal patent specification; it’s the technical invention disclosure wording I would take to your company’s patent/IP team.

⸻

13. The core inventive concept

If I were preparing your internal invention disclosure, I’d highlight these five elements:



1. Distributed Health Model

SOM understands the health of:

SOM
Wi-Fi
Network
MQTT
RPCA
Master
CAN
A
B
C

2. Dependency-Aware Fault Isolation
Failure
 ↓
Check dependencies
 ↓
Separate root cause from symptoms


3. Hierarchical Minimal-Impact Recovery

Retry
 ↓
Reconnect
 ↓
Restart service
 ↓
Reset interface
 ↓
Reset component
 ↓
Reboot

4. Autonomous Recovery Validation

Recovery
 ↓
Functional validation
 ↓
Success → Healthy
Failure → Escalate

5. Persistent Fault Correlation

Repeated failures
       ↓
Transient → Persistent
       ↓
Escalated recovery / diagnostic state















PATENT INVENTION DISCLOSURE

1. Proposed Title

Autonomous Runtime Fault Detection, Root-Cause Isolation and Hierarchical Self-Healing Framework for Distributed Embedded Systems

Alternative shorter title

Dependency-Aware Self-Healing Framework for Distributed Embedded Systems

⸻

2. Technical Field

The invention relates to embedded systems, distributed embedded architectures, runtime health monitoring, fault detection, communication management, and autonomous system recovery.

More particularly, the invention relates to a system-on-module (SOM)-based framework that continuously monitors the runtime health of multiple software, communication, controller, and embedded-board components, identifies probable root causes of failures using deterministic rules and component dependencies, automatically performs an appropriate hierarchical recovery operation, and validates whether the recovery was successful.

⸻

3. Background / Problem Statement

Modern embedded products commonly consist of multiple interconnected controllers and communication subsystems. A typical system may contain a system-on-module (SOM), a master controller, multiple subordinate embedded boards, wireless/network connectivity, application services, MQTT or other communication protocols, RPC/RPCA interfaces, and CAN communication.

Failures in such systems can occur at different levels. Examples include:

* Wi-Fi disconnection
* Network interface failure
* Server connection failure
* MQTT session failure
* MQTT publish/subscribe failure
* RPC/RPCA communication timeout
* Master-board communication failure
* CAN communication failure
* Individual CAN-node failure
* Application-process crash
* Software service termination
* Software task becoming unresponsive
* Communication timeout
* Repeated transient failures
* Resource exhaustion or abnormal runtime conditions

In conventional systems, recovery is frequently implemented using isolated mechanisms such as watchdog timers, fixed retries, service restarts, or complete device reboot.

Such mechanisms may not determine the actual root cause of the failure. Consequently, a failure in one subsystem may result in unnecessary restarting of unrelated subsystems or the complete embedded product.

For example, an MQTT failure may actually be caused by a Wi-Fi failure. Similarly, an MQTT failure may be caused by a server-side connectivity problem, while an RPC failure may be caused by a master-board failure.

There is therefore a need for a unified runtime framework capable of understanding dependencies between system components, identifying the probable source of a failure, applying the least disruptive appropriate recovery action, and validating the result before escalating the recovery operation.

⸻

4. Summary of the Invention

The proposed invention provides an autonomous runtime health and self-healing framework implemented primarily on a system-on-module (SOM).

The SOM operates as a supervisory communication and health-management node for a distributed embedded system.

The distributed system may comprise:

* SOM
* Wi-Fi/network subsystem
* MQTT or equivalent communication service
* Application software
* RPC/RPCA communication
* Master Board
* CAN bus
* Multiple CAN-connected subordinate boards

The SOM continuously collects health information from the different components.

When a fault or abnormal operating condition is detected, the framework evaluates the dependency relationship between components to determine a probable root cause rather than treating every downstream symptom as an independent failure.

The framework then selects a recovery operation according to a predefined deterministic recovery policy.

The recovery operation may be progressively escalated from a low-impact action to a higher-impact action.

After each recovery operation, the framework performs an independent validation of the affected functionality.

If the functionality is restored, the system returns the component to a healthy state.

If recovery fails, the framework escalates to the next appropriate recovery level.

The framework can therefore provide autonomous runtime self-healing without requiring artificial intelligence, machine learning, cloud-based decision making, or manual intervention for predefined fault conditions.

⸻

5. System Architecture

The proposed system can be represented as follows:

                         SERVER / CLOUD
                              |
                       MQTT / NETWORK
                              |
                    +---------v---------+
                    |       SOM         |
                    |                   |
                    | Runtime Health    |
                    | Management Engine  |
                    |                   |
                    | +---------------+ |
                    | | Health Monitor | |
                    | +-------+-------+ |
                    |         |         |
                    | +-------v-------+ |
                    | | Fault         | |
                    | | Detector      | |
                    | +-------+-------+ |
                    |         |         |
                    | +-------v-------+ |
                    | | Root-Cause    | |
                    | | Isolation     | |
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
                    +----+----+----+----+
                         |    |    |
                        CAN  CAN  CAN
                         |    |    |
                       +--+  +-+  +-+
                       | A|  | B|  | C|
                       +---+ +---+ +---+

⸻

6. Major Components

6.1 Runtime Health Monitor

The Runtime Health Monitor continuously monitors the operational state of system components.

Health indicators may include:

* Heartbeat messages
* Response time
* Connection state
* Communication timeout
* Process status
* Service status
* CAN response
* RPC/RPCA response
* Network state
* MQTT session state
* Application state
* Resource utilization
* Fault counters

Each monitored component may be represented by a health state such as:

HEALTHY
DEGRADED
FAILED
RECOVERING
UNKNOWN

⸻

6.2 Fault Detection Engine

The Fault Detection Engine identifies abnormal runtime conditions based on deterministic conditions.

Examples include:

Heartbeat timeout
Response timeout
Connection lost
Unexpected process termination
Repeated communication failure
CAN node unavailable
MQTT session disconnected
Network interface unavailable
RPC/RPCA timeout

The system does not necessarily treat the first detected failure as a permanent fault. Retry counts, timeout thresholds, and historical failure information may be used to distinguish transient conditions from persistent failures.

⸻

7. Dependency-Aware Root-Cause Isolation

A key aspect of the invention is a dependency model describing relationships between system components.

For example:

Server
   |
   v
Wi-Fi
   |
   v
Network
   |
   v
MQTT
   |
   v
Application
   |
   v
RPC/RPCA
   |
   v
Master Board
   |
   v
CAN Bus
  /|\
 / | \
A  B  C

When a downstream component reports a failure, the framework checks the health of its dependencies before selecting a recovery operation.

For example:

MQTT failure detected
        |
        +-- Wi-Fi healthy?       YES
        |
        +-- Network healthy?     YES
        |
        +-- Server reachable?    YES
        |
        +-- MQTT session?        FAILED

The framework therefore identifies an MQTT session failure as the probable fault rather than unnecessarily restarting the complete device.

⸻

8. Hierarchical Recovery Mechanism

The framework maintains predefined recovery levels.

An example recovery hierarchy is:

Level 0:
Continue monitoring
Level 1:
Retry failed operation
Level 2:
Reconnect affected communication session
Level 3:
Restart affected software service/process
Level 4:
Reset affected communication interface
Level 5:
Reset affected controller/component
Level 6:
Restart affected subsystem
Level 7:
Controlled SOM reboot
Level 8:
System-level recovery

The actual recovery levels may differ depending on the hardware and application.

The framework selects the lowest-impact recovery operation capable of addressing the identified fault.

⸻

9. Recovery Validation

A recovery operation is not considered successful merely because the recovery command completed.

The framework performs a separate validation procedure.

For example:

MQTT Failure
     |
     v
Reconnect MQTT
     |
     v
Connection established?
     |
     v
Publish/Subscribe validation
     |
     v
Expected response received?
     |
   +---+---+
   |       |
  YES      NO
   |       |
HEALTHY   Escalate

The validation mechanism can therefore determine whether the original functionality has actually been restored.

⸻

10. Example Scenario 1 — MQTT Failure

The SOM detects that the MQTT heartbeat has exceeded a predefined timeout.

The framework performs:

1. Detect MQTT heartbeat timeout
2. Verify Wi-Fi status
3. Verify network connectivity
4. Verify server reachability
5. Verify MQTT session
6. Classify probable MQTT failure
7. Attempt MQTT reconnection
8. Validate MQTT communication

If validation succeeds:

MQTT = HEALTHY

If validation fails repeatedly:

Restart MQTT service
        |
        v
Validate
        |
        v
Reset network interface
        |
        v
Validate
        |
        v
Escalate if required

⸻

11. Example Scenario 2 — Wi-Fi Failure

The SOM detects that the network interface has become unavailable.

The framework may execute:

Detect Wi-Fi failure
       |
       v
Check Wi-Fi interface
       |
       v
Restart Wi-Fi subsystem
       |
       v
Reassociate with configured network
       |
       v
Validate IP connectivity
       |
       v
Validate server connectivity
       |
       v
Reconnect MQTT
       |
       v
Validate complete communication path

The framework avoids restarting unrelated CAN or application components unless required.

⸻

12. Example Scenario 3 — Master Board Failure

The SOM detects that the Master Board heartbeat or RPC/RPCA response has exceeded a predefined timeout.

The framework may perform:

RPCA timeout
    |
    v
Retry RPCA request
    |
    v
Check Master Board heartbeat
    |
    v
Diagnose Master Board communication
    |
    v
Perform configured Master Board recovery
    |
    v
Wait for Master Board initialization
    |
    v
Validate Master Board
    |
    v
Validate CAN communication

If recovery fails, the SOM can escalate according to the configured recovery policy.

⸻

13. Example Scenario 4 — Individual CAN Board Failure

Assume Boards A, B, and C are connected to the Master Board.

The system detects:

Board A = HEALTHY
Board B = NO RESPONSE
Board C = HEALTHY

Because A and C continue responding, the framework can determine that the CAN bus is likely operational and isolate the failure to Board B or its communication path.

The framework may then:

Detect Board B timeout
        |
        v
Retry Board B communication
        |
        v
Send diagnostic request
        |
        v
Attempt configured Board B recovery
        |
        v
Validate Board B

If Board B remains unavailable, the framework can place Board B into a defined failed/degraded state while allowing unaffected components to continue operating where permitted by the product’s safety and functional requirements.

⸻

14. Fault History and Repeated Failure Detection

The framework may maintain a persistent or runtime fault history containing:

* Fault identifier
* Component
* Timestamp
* Fault type
* Detected symptoms
* Probable root cause
* Recovery action
* Recovery result
* Recovery duration
* Number of occurrences

For example:

Fault:
MQTT Session Failure
Occurrence:
5 times within 30 minutes
Recovery:
Reconnect
Result:
Successful
Classification:
Repeated / Persistent Fault

If the same failure repeatedly occurs, the framework may escalate the recovery strategy or place the system into a degraded diagnostic state.

⸻

15. Deterministic Rule Engine

The proposed framework can operate without AI or machine learning.

Example:

IF
MQTT heartbeat timeout
AND
Wi-Fi healthy
AND
network reachable
AND
server reachable
THEN
Classify as MQTT session failure
AND
Execute MQTT reconnect recovery.

Another example:

IF
CAN node B timeout
AND
CAN node A healthy
AND
CAN node C healthy
AND
Master Board healthy
THEN
Classify probable fault as CAN node B
AND
Execute node-specific recovery.

The deterministic approach provides predictable and testable recovery behavior.

⸻

16. Key Advantages

The proposed invention can provide the following advantages:

1. Autonomous runtime fault recovery.
2. Reduced unnecessary full-device reboots.
3. Root-cause-oriented fault isolation.
4. Dependency-aware failure analysis.
5. Least-disruptive recovery selection.
6. Automatic escalation when recovery fails.
7. Independent validation of recovery success.
8. Improved system availability.
9. Reduced manual service intervention.
10. Support for distributed embedded architectures.
11. Local operation without mandatory cloud or AI dependency.
12. Ability to isolate failures to individual communication or hardware components.
13. Historical identification of repeated runtime failures.
14. Adaptability to different SOMs, master controllers, and subordinate boards.

⸻

17. Potential Novelty / Inventive Features

The invention may particularly emphasize the combination of:

A. Dependency-aware runtime health monitoring

The SOM maintains relationships between communication and system components rather than monitoring each component independently.

B. Root-cause isolation based on dependency health

The framework uses the health state of upstream and downstream components to distinguish a primary failure from secondary symptoms.

C. Least-disruptive hierarchical recovery

The system progressively escalates recovery operations rather than immediately rebooting the entire product.

D. Independent post-recovery validation

Each recovery action is followed by functional validation before the component is declared healthy.

E. Distributed embedded-system recovery

The SOM can supervise and recover communication paths spanning:

Network
→ MQTT
→ RPC/RPCA
→ Master Board
→ CAN
→ Individual embedded boards

F. Persistent failure correlation

Repeated occurrences of the same failure can influence subsequent recovery and escalation behavior.

⸻

18. Proposed Independent Claim Concept

The following is a technical starting point for the patent team and is not intended to replace legal claim drafting:

A system for autonomous runtime self-healing of a distributed embedded system, comprising:

a system-on-module configured to communicate with a master controller and a plurality of subordinate embedded controllers;

a plurality of communication interfaces including at least one wireless/network interface and at least one controller-area-network interface;

a runtime health monitoring module configured to obtain health indicators from a plurality of software, communication, controller, and embedded-board components;

a dependency model representing relationships among the plurality of components;

a fault detection and classification module configured to identify an abnormal runtime condition and determine a probable fault component based at least in part on the health indicators and the dependency model;

a recovery decision module configured to select a recovery operation from a plurality of hierarchical recovery operations based on the identified fault component and a predefined recovery policy;

a recovery execution module configured to execute the selected recovery operation; and

a recovery validation module configured to determine whether the functionality associated with the identified fault has been restored following execution of the recovery operation, wherein failure of validation causes the recovery decision module to select a subsequent recovery operation having a higher recovery level.

⸻

19. Possible Dependent Claim Concepts

The patent team may consider dependent claims covering:

1. MQTT session failure detection and recovery.
2. Wi-Fi interface recovery.
3. Network connectivity validation.
4. Server reachability validation.
5. RPC/RPCA communication recovery.
6. Master Board health monitoring and recovery.
7. CAN bus health monitoring.
8. Individual CAN-node fault isolation.
9. Individual CAN-node recovery.
10. Heartbeat-based fault detection.
11. Timeout-based fault detection.
12. Recovery retry thresholds.
13. Progressive recovery escalation.
14. Post-recovery functional validation.
15. Fault-history-based escalation.
16. Persistent versus transient fault classification.
17. Dependency graph/model-based root-cause isolation.
18. Continued operation of unaffected components after localized failure.
19. SOM reboot as a final recovery level.
20. Hardware-independent or configurable recovery policies.

⸻

20. Example End-to-End Operation

             RUNTIME OPERATION
                    |
                    v
            Health Monitoring
                    |
                    v
             Fault Detected
                    |
                    v
          Check Dependencies
                    |
                    v
          Isolate Probable Cause
                    |
                    v
          Select Recovery Level
                    |
                    v
            Execute Recovery
                    |
                    v
          Validate Functionality
               /          \
             PASS          FAIL
              |              |
              v              v
          HEALTHY       Escalate Recovery
                             |
                             v
                       Execute Next Level
                             |
                             v
                          Validate

⸻

21. Example Implementation Platform

A prototype implementation may be developed using:

* Embedded Linux
* C/C++
* SOM such as NXP i.MX6-based hardware
* Future SOM platforms such as Variscite-based systems
* Wi-Fi subsystem
* MQTT client
* RPC/RPCA communication
* CAN communication
* Linux process/service monitoring
* Heartbeat mechanisms
* Watchdog mechanisms
* Configurable recovery policies
* Runtime diagnostic logging

The framework can be designed to remain independent of a specific SOM processor so that the architecture can subsequently be deployed on different hardware platforms.

⸻

22. Important Distinction From Conventional Watchdog Recovery

A conventional watchdog generally determines that a system or task has stopped responding and performs a predefined reset.

The proposed framework instead performs:

Detect
   ↓
Correlate
   ↓
Determine probable root cause
   ↓
Select minimum-impact recovery
   ↓
Execute recovery
   ↓
Validate recovery
   ↓
Escalate if required

Therefore, the framework is intended to provide context-aware and dependency-aware runtime recovery, rather than merely resetting a non-responsive system.

⸻

23. Suggested One-Line Invention Statement

“A SOM-based autonomous self-healing framework that monitors a distributed embedded system, isolates probable runtime fault causes using component dependencies, performs progressively escalating minimum-impact recovery actions, and independently validates recovery before escalation.”

⸻

24. Keywords

Embedded System; SOM; Self-Healing; Autonomous Recovery; Runtime Health Monitoring; Fault Detection; Root Cause Isolation; Dependency Graph; MQTT; Wi-Fi; RPC; RPCA; CAN; Master Controller; Distributed Embedded System; Watchdog; Service Recovery; Fault Classification; Recovery Validation; Hierarchical Recovery; Embedded Linux; Communication Fault; Runtime Diagnostics.


   

