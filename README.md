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

