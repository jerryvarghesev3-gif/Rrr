GUI Ticket 1 — Landing Page to Product Selection Flow

Purpose

Handle the first visible user flow from app launch to product selection.

Tasks

Task 1 — Create Landing Page QML layout

Definition
Create the initial landing page UI with entry point for firmware upgrade flow.

Acceptance Criteria
	•	Landing page loads as the first screen
	•	Page layout is stable and matches application theme
	•	Firmware upgrade entry action is visible

Task 2 — Create Product Selection Page QML layout

Definition
Create the product selection page UI that appears after landing page.

Acceptance Criteria
	•	Product selection page is rendered correctly
	•	Product options are visible in UI
	•	Layout supports future product list updates

Task 3 — Implement Landing → Product Selection navigation

Definition
Implement screen transition from landing page to product selection page.

Acceptance Criteria
	•	Navigation occurs correctly from landing page
	•	No UI flicker or broken state during transition
	•	Back navigation returns to landing page

Task 4 — Bind selected product to GUI session state

Definition
Store selected product in GUI-side session state for use in next screens.

Acceptance Criteria
	•	Selected product is retained after navigation
	•	Product context is accessible to next page
	•	Changing selection updates stored state correctly

⸻

GUI Ticket 2 — Firmware Upgrade Start to AppData Analysis Flow

Purpose

Handle the flow after product selection where firmware upgrade starts and the UI moves into AppData analysis.

Tasks

Task 1 — Create Firmware Upgrade Starting Page QML layout

Definition
Create a page that indicates upgrade workflow is starting after product selection.

Acceptance Criteria
	•	Page displays workflow start state clearly
	•	User sees transition from selection to startup phase
	•	Layout supports status/progress messages

Task 2 — Create AppData Analysis Status Page QML layout

Definition
Create UI page to show AppData analysis progress such as bundle lookup, unzip check, and file availability.

Acceptance Criteria
	•	AppData analysis page is visible
	•	Page supports dynamic status messages
	•	Layout is ready for success/failure state display

Task 3 — Implement Product Selection → Firmware Start → AppData Analysis navigation

Definition
Implement page flow from selected product into upgrade-start screen and then into AppData analysis screen.

Acceptance Criteria
	•	Correct sequential navigation is followed
	•	Product context is preserved across pages
	•	No invalid direct jump to later pages occurs

Task 4 — Implement AppData analysis success/failure visual state handling

Definition
Show UI state for AppData analysis result before launcher start.

Acceptance Criteria
	•	Success state is visibly different from failure state
	•	Failure state blocks forward navigation
	•	Success state allows next workflow transition

⸻

GUI Ticket 3 — Launcher Start and Device Analysis Flow

Purpose

Handle the UI flow after AppData success, where launcher starts and device/product information is read.

Tasks

Task 1 — Create Launcher Started Page QML layout

Definition
Create page that shows launcher startup is in progress or completed.

Acceptance Criteria
	•	Launcher status page renders correctly
	•	Page supports startup/loading indication
	•	Page supports ready/failure status display

Task 2 — Create Configs and Versions Reading Page QML layout

Definition
Create page to show launcher-driven reading of configs, hardware information, and software versions.

Acceptance Criteria
	•	Reading page is displayed correctly
	•	UI supports multiple reading states
	•	Page can show activity without user interaction

Task 3 — Implement AppData Success → Launcher Started → Config Reading navigation

Definition
Implement navigation sequence from successful AppData stage into launcher startup page and then into config/version reading page.

Acceptance Criteria
	•	Launcher page appears only after AppData success
	•	Config reading page appears only after launcher start
	•	No premature transition happens

Task 4 — Bind device read results to analysis screen placeholders

Definition
Bind read outputs such as hardware config, software version, and detected product info to the GUI display model.

Acceptance Criteria
	•	Read values can be shown in UI
	•	Empty and loading states are handled
	•	Updated values appear without reopening the page

⸻

GUI Ticket 4 — Manifest / Matrix Analysis and Rule Generation Flow

Purpose

Handle the GUI flow where launcher compares device/config data with manifest.xml and compatibility_matrix.json and shows rule-generation progress.

Tasks

Task 1 — Create Package and Rule Analysis Page QML layout

Definition
Create page showing analysis of manifest.xml, binaries, and compatibility matrix.

Acceptance Criteria
	•	Analysis page renders correctly
	•	Page supports stepwise status display
	•	UI can show matrix/rule evaluation state

Task 2 — Create Rule Generation Result Section in QML

Definition
Create UI section to show generated rule result or upgrade strategy summary.

Acceptance Criteria
	•	Rule generation result area is visible
	•	UI supports summary of derived upgrade path
	•	Empty/loading/result states are handled

Task 3 — Implement Config Reading → Manifest/Matrix Analysis → Rule Generation navigation

Definition
Implement automatic flow from config/version reading screen into package/rule analysis and then to rule generation result state.

Acceptance Criteria
	•	Navigation follows correct sequence
	•	Analysis page is shown before rule result is shown
	•	No screen ordering mismatch occurs

Task 4 — Implement analysis failure and blocked-flow UI handling

Definition
Handle GUI behavior when manifest or matrix analysis fails and upgrade rule cannot be generated.

Acceptance Criteria
	•	Failure state is clearly shown
	•	Workflow does not continue to upgrade-ready state
	•	User sees blocked reason in UI

⸻

GUI Ticket 5 — Upgrade Ready / Board-by-Board Start Flow

Purpose

Handle the final GUI transition from generated rules into board-by-board upgrade start.

Tasks

Task 1 — Create Upgrade Ready Summary Page QML layout

Definition
Create page showing that rule generation is complete and board upgrade workflow is ready to start.

Acceptance Criteria
	•	Upgrade-ready page displays correctly
	•	Page can show selected product, detected version, and chosen rule summary
	•	UI clearly indicates next stage is board upgrade

Task 2 — Create Board Upgrade Progress Container Page

Definition
Create QML page/container that will host board-by-board upgrade progress states.

Acceptance Criteria
	•	Progress container page is available
	•	UI supports multiple board entries
	•	Layout supports sequential upgrade visualization

Task 3 — Implement Rule Generation Result → Upgrade Ready → Board Progress navigation

Definition
Implement transition from rule generation completion into upgrade-ready page and then into board progress page.

Acceptance Criteria
	•	Upgrade-ready page appears only after valid rule generation
	•	Board progress page starts only from valid ready state
	•	Navigation remains sequential and stable

Task 4 — Bind upgrade stage labels to board workflow states

Definition
Bind board upgrade stage labels such as waiting, running, completed, failed to GUI state model.

Acceptance Criteria
	•	Board states are visible in UI
	•	State labels update correctly
	•	UI supports multiple board transitions

