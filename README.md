Now we have the architecture clearly defined. Your existing Service Tool architecture can become the foundation rather than creating a completely separate simulator.

The key difference is:

Normal mode: BAS → extract → transfer firmware to physical bed.
Simulation mode: BAS → extract → load into virtual device model → execute simulated RPCA/API behavior.

1. Existing architecture

From what you’re describing:

Kotlin Service Tool
        │
        ▼
       JNI
        │
        ▼
       API
        │
        ▼
    RPCA Method
        │
        ▼
   Physical Board
        │
        ▼
    Centrella Bed


    For example:

    Kotlin
  │
  │ upgradeFirmware()
  ▼
JNI
  │
  ▼
Native API
  │
  ▼
RPCA
  │
  ▼
Physical Bed



2. New Simulation Mode

We introduce a mode switch at the native/service layer:

                  Kotlin
                    │
                    ▼
                   JNI
                    │
                    ▼
              Service API
                    │
             ┌──────┴──────┐
             │             │
         REAL MODE     SIMULATION MODE
             │             │
          RPCA          Simulator
             │             │
       Physical Bed    Virtual Bed



       This is important because Kotlin doesn’t need to know all the simulation details.

       The Kotlin layer can simply say:

       Simulation = ON


       and the native layer routes the operation appropriately.

      3. BAS loading happens in Kotlin

Yes.

The user could do:


Service Tool
     │
     ▼
Firmware Upgrade
     │
     ▼
Select BAS
     │
     ▼
Download/Documents
     │
     ▼
Centrella_xxx.bas

In normal mode:

BAS
 ↓
Extract
 ↓
Find required firmware
 ↓
Transfer to physical board
 ↓
Upgrade

In simulation mode:

BAS
 ↓
Extract
 ↓
Analyze package
 ↓
Create virtual board state
 ↓
Load firmware/configuration
     into simulator model
 ↓
Simulation ready

We don’t transfer anything to the physical bed.

That’s the key difference.

⸻

4. But “extract” and “execute” are two different things

This is something we should design carefully.

Suppose BAS contains:

acb.bin
atlas.bin
dcb.bin
hfb.bin
mcb.bin
scr.bin
...

The simulator first does:

BAS
 ↓
Extraction
 ↓
Package Manager
 ↓
Board artifacts

Then:

acb.bin     → ACB simulation model
atlas.bin   → ATLAS simulation model
dcb.bin     → DCB simulation model
...


We should not assume that simply loading the .bin means we’re executing the firmware.

For POC-1, we can use the binaries as firmware/package identity and configuration inputs, while the simulator implements the board behavior.

Later we can investigate actually executing/emulating selected firmware.

⸻

5. Your RPCA idea fits beautifully

   Suppose the existing service operation is:

   Kotlin
   ↓
JNI
   ↓
API
   ↓
RPCA
   ↓
Physical Board

We can introduce:

Kotlin
   ↓
JNI
   ↓
API
   ↓
RPCA Interface
   ↓
┌───────────────────────┐
│                       │
▼                       ▼
Real RPCA           Simulation RPCA
│                       │
▼                       ▼
Physical board      Virtual board


So if the Service Tool invokes:

getBedStatus()


the application doesn’t need to care whether it is:
REAL

or:

SIMULATION

The backend provides the appropriate response.

⸻

6. Example: Firmware Upgrade

   Real mode

   Kotlin
  │
  │ Select BAS
  ▼
Extract BAS
  │
  ▼
Find board firmware
  │
  ▼
JNI
  │
  ▼
API
  │
  ▼
RPCA
  │
  ▼
Physical board
  │
  ▼
Firmware transferred


Simulation mode

Kotlin
  │
  │ Select BAS
  ▼
Extract BAS
  │
  ▼
Identify board firmware
  │
  ▼
Simulation Manager
  │
  ├── ACB version = X
  ├── ATLAS version = Y
  ├── DCB version = Z
  ├── SOM version = A
  └── etc.
  │
  ▼
Virtual Centrella


The simulator can then report:

Firmware Package
────────────────────────

ACB       1.2.4     ✓ Loaded
ATLAS     3.1.2     ✓ Loaded
DCB       2.4.1     ✓ Loaded
HFB       1.7.0     ✓ Loaded
MCB       2.1.5     ✓ Loaded
SOM       4.3.2     ✓ Loaded

Simulation Environment
────────────────────────

✓ Package Valid
✓ All required boards present
✓ Version compatibility OK

SIMULATION READY

7. Then each Service Tool feature can be simulated

   For example:

                   Kotlin
                   │
                  JNI
                   │
               Native API
                   │
             Simulation API
                   │
      ┌────────────┼────────────┐
      │            │            │
    RPCA         CAN          SOM
   Model        Model         Model
      │            │            │
      └────────────┼────────────┘
                   │
             Virtual Bed


   Features could include:

   Firmware Upgrade
Device Information
Configuration
Calibration
Diagnostics
Board Status
CAN Operations
SOM Operations
Wi-Fi configuration
MQTT configuration
DeviceBridge


Each feature can have a corresponding simulation implementation.

⸻

8. This is where your existing RPCA architecture helps

   You don’t want:

   Kotlin
 ├── Real firmware implementation
 ├── Simulation firmware implementation
 ├── Real configuration
 ├── Simulation configuration
 ├── Real diagnostics
 └── Simulation diagnostics



That would become messy.

Instead:
Kotlin
    │
    ▼
JNI
    │
    ▼
Common API
    │
    ▼
RPCA abstraction
    │
    ├──────────────┐
    │              │
 REAL backend   SIM backend
    │              │
    ▼              ▼
Physical       Virtual
Centrella      Centrella


The API contract stays common.

That’s a major architectural advantage.

9. I would also add a Simulation State Database

    Once BAS is loaded:

   Simulation State

Product:
Centrella

BAS:
1.47.305

ACB:
1.2.4

ATLAS:
3.1.2

DCB:
2.4.1

SOM:
4.3.2

Wi-Fi:
Connected

MQTT:
Connected

DeviceBridge:
Disconnected

Battery:
85%

Bed Height:
45 cm

Head:
30°

Foot:
15°

Then each RPCA simulation method modifies or reads this state.

For example:

RPCA_GET_BED_STATUS()
        ↓
Simulation State
        ↓
return:
height = 45
head = 30
foot = 15

10. And eventually the simulator can become a virtual bed

initially:

    Kotlin
 ↓
Simulation State
 ↓
Logs


Then:

Kotlin
 ↓
Simulation State
 ↓
2D bed visualization


Then eventually:

Kotlin
 ↓
Simulation State
 ↓
3D Centrella



So 3D isn’t required for the first POC.

⸻

11. This also gives you the second simulation path later

You mentioned:

If we don’t have BAS, whatever we build with Submodule we can simulate.

Exactly.

We can eventually have:

                 Simulation Manager
                        │
              ┌─────────┴──────────┐
              │                    │
          BAS Adapter          Build Adapter
              │                    │
         Existing BAS        Genesis/SomApp
              │                    │
              └─────────┬──────────┘
                        ▼
                 Simulation API
                        │
                        ▼
                 Virtual Centrella



  So BAS is the first input source, while a Genesis/SomApp simulation build becomes the second input source later.

⸻

I would define your POC now as this

Centrella Service Tool – BAS-Based Bed Simulation

Input:

Centrella BAS package

Processing:

Kotlin
 ↓
BAS selection
 ↓
BAS extraction
 ↓
BAS validation
 ↓
Board/package identification
 ↓
Simulation state creation

Execution:

Service API
 ↓
JNI
 ↓
RPCA abstraction
 ↓
Simulation backend
 ↓
Virtual Centrella

No physical bed required.

And importantly, we’re not trying to emulate the entire Centrella hardware on day one. We’re creating a virtual representation capable of responding to the Service Tool’s existing API/RPCA operations.

That makes the POC much more achievable.

The first technical milestone I’d target

Select BAS
     ↓
Extract successfully
     ↓
Read manifest
     ↓
Identify all board binaries
     ↓
Create simulation session
     ↓
Invoke ONE existing RPCA feature
     ↓
Receive simulated response
     ↓
Display result in Kotlin

If we can demonstrate one real Service Tool feature going from Kotlin → JNI → API → RPCA abstraction → simulated board → response, you’ve proven the core architecture. Then we can systematically add the other features.

