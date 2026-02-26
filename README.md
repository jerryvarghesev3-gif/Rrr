Start
  ↓
User Selects Product (Dynamo / Centrella)
  ↓
Request USB Permission
  ↓
USB Device Detected?
  ├── No → Show Popup:
  │        "No USB device detected"
  │        ↓
  │       End
  │
  └── Yes
        ↓
Get Product Model from USB
        ↓
Identify Product Type
(P80 → Dynamo
 P79 → Centrella)
        ↓
Is Product Matching Selected App?
        ├── No
        │     ↓
        │ Show Popup:
        │ "Wrong product connected"
        │     ↓
        │ Hard Stop & Disconnect
        │     ↓
        │ Return to Home Screen
        │
        └── Yes
              ↓
Connect to Device (MCB)
              ↓
Load Properties
              ↓
Start Board Collection
              ↓
Start Bed Status Poll
              ↓
Load UI Data
              ↓
Success
