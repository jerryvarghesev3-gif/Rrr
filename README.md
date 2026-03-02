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
