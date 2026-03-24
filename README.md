Description

Initialize the launcher core runtime by implementing application startup flow, configuration loading, and environment setup.

This task ensures that the launcher application (plupdater / launcher core) correctly initializes all required components during startup, including configuration management, logging setup, and preparation of the runtime environment for backend integration.

This subtask focuses on:
	•	Application startup bootstrap (main entry flow)
	•	Configuration file loading and validation
	•	Initialization of application environment (paths, dependencies, runtime context)
	•	Basic logging setup for diagnostics

It does not include:
	•	Device communication logic
	•	GUI-backend integration
	•	Version comparison or upgrade workflow

⸻

Acceptance Criteria
	•	Launcher application starts successfully without runtime errors
	•	Application configuration is loaded from defined source (file/env/default)
	•	Configuration values are validated and accessible across the application
	•	Application environment is initialized, including:
	•	Required directories / paths
	•	Runtime dependencies
	•	Environment variables (if applicable)
	•	Logging mechanism is initialized during startup
	•	Errors during configuration or environment setup are handled gracefully
	•	Startup sequence completes and application is ready for further module integration

⸻

Output
	•	Startup bootstrap flow implemented for launcher core
	•	Configuration management system initialized and validated
	•	Application environment setup completed during startup
	•	Logging initialized and available for debugging and tracing
	•	Centralized initialization logic established for future modules
	•	Launcher is able to:
	•	Start reliably
	•	Load configuration
	•	Prepare runtime context for backend and communication layers

