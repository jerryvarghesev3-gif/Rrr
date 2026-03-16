What Went Wrong – Dynamo Master Migration

1. Initial Branch Setup
	•	Initially, I was working on the Dynamo branch.
	•	Based on Bryan’s suggestion, I created a new branch called Dynamo_Master_Migration to support the migration process.

2. Long-Term Development on feature/rc2
	•	I had been working on the feature/rc2 branch for more than one year.
	•	In November 2025, Bryan requested that I merge this branch with the master branch and resolve conflicts between:
	•	rc2/Centrella new features
	•	master branch updates
	•	This resulted in hundreds of merge conflicts.
	•	I attempted to resolve these conflicts, including some manual conflict resolutions, which later contributed to inconsistencies.

3. Environment Disruption During UI Implementation
	•	In October, I received the Universal Remote UI implementation from Suresh.
	•	I spent approximately two days reconfiguring the Qt and VM environments to support Designer Studio.
	•	During this process:
	•	The VM configuration was disrupted.
	•	I was able to recover approximately 95% of the configuration.
	•	However, Qt Creator issues remained, including:
	•	Absolute vs. relative path problems
	•	CMake configuration issues

4. Shift in Priorities
	•	While I was still resolving configuration and merge issues, I received new feature requirements from Bryan in mid-November.
	•	During December, my focus shifted primarily to implementing these features.

5. Branch Review and Recommendation
	•	In November, the Dynamo_Master_Migration branch was reviewed by Jason and Ed.
	•	Jason recommended:
	•	Avoid manual conflict resolution
	•	Perform a complete automatic merge
	•	Ignore the current migration branch
	•	Start a fresh migration from master

6. Final Resolution
	•	Based on this recommendation:
	•	I restarted the migration using the original Dynamo branch.
	•	I completed a full fresh migration with the master branch.








1. VM Configuration and Parallel Debugging (December)
	•	In December, Veeru was assigned TCO-related work, so I shared my VM image (zip file) with him.
	•	Both of us encountered the same Qt Creator relative-path configuration issues.
	•	While Veeru was investigating the environment issue, I continued working in parallel on:
	•	Fixing configuration problems
	•	Implementing required features
	•	This allowed progress to continue despite the environment challenges.

2. Build Issues Identified (January)
	•	In January, Jason checked out the Dynamo branch and attempted to build the project.
	•	During the build process, he encountered:
	•	Submodule configuration issues
	•	CMakeLists configuration problems

3. Issue Investigation and Resolution
	•	We worked together to investigate the issue.
	•	Initially, the root cause was not clear.
	•	Jason shared screenshots of his build configuration.
	•	After comparing configurations, we identified missing configuration settings, which allowed us to resolve the issue successfully.

4. Process Deviation (Lessons Learned)
	•	After completing the new migration, some minor patch fixes were merged directly into the branch.
	•	These changes were made without following the standard migration or merge process, which was an oversight on my part.
	•	This has been identified as a process improvement area going forward.

5. Requirement Clarity Challenges
	•	At the beginning of the project, development work was based on the SRS requirements.
	•	Over time, formal requirements were not consistently provided.
	•	As a result, development on the Dynamo project continued without a clearly defined or updated requirement specification, which made prioritization and planning more difficult.











1. Sprint Planning and Workload Management
	•	During development of the Service Tool, the sprint planning process did not follow a typical Agile/Scrum incremental approach.
	•	A large number of tickets (representing hundreds of story points) were placed into active sprints simultaneously.
	•	I suggested planning work incrementally per sprint (for example, 10–20 story points per two-week sprint) to allow better focus and tracking.
	•	However, all tickets remained active so that overall progress could be monitored.
	•	This resulted in:
	•	Increased workload pressure
	•	Difficulty prioritizing tasks
	•	Extended working hours to manage the scope

2. Limited System Testing Availability
	•	From the early stages of development, I requested support for testing the application on target hardware (MCB & SOM).
	•	At that time, the hardware environment was still not stable, which limited full system-level testing.
	•	As a result:
	•	Testing was mostly limited to developer-level testing.
	•	Full validation of the Service Tool was delayed.
	•	This lack of complete testing contributed to some of the issues identified later.

3. Migration and Testing Gaps
	•	During the re-migration of master into the Dynamo branch, a large number of conflicts had to be resolved.
	•	Most functionality was tested after the migration.
	•	However, due to the size and complexity of the changes:
	•	Some working scenarios and code paths were unintentionally missed during testing.
	•	This has been identified as an area for improvement in future migrations.








Create a professional flowchart for an Automated Firmware Updater.

Start with App.exe containing bundle.zip and launcher.exe.

Extract bundle.zip to AppData.

Decision: Was unzip successful?

If no, check if required files exist in AppData.

If files do not exist, show error popup "Unable to unzip or files not found".

If files exist or unzip succeeds, start launcher.exe.

launcher.exe reads hardware configuration, firmware version, bootloader version, and device configuration.

Store this information in a device data structure.

Load compatibility matrix from AppData.

Determine transfer method and upgrade rules.

Generate upgrade plan.

Identify boards such as ICB, HIB, WAM.

Upgrade boards sequentially.

Verify firmware version after each update.

End with upgrade completion.
