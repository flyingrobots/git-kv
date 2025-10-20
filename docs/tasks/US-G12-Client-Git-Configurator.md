# Task: Client Git Configurator

**Feature:** `US-G12 — Discovery on Fresh Clone`

## 1. Description

Implement the client-side logic responsible for modifying the local Git configuration. This module will be used by `git kv remote setup` to set the `origin.pushurl`.

## 2. Acceptance Criteria

- A function `ConfigureRemote(remoteName string, key string, value string) error` is defined.
- On success, it returns `nil`.
- On failure, it returns a `ConfigError` object/enum containing:
  - `Code`: machine-friendly code (e.g., `INVALID_REMOTE`, `GIT_FAILED`, `PERMISSION_DENIED`).
  - `Message`: human-readable message.
  - `ExitCode`: underlying `git` exit code or OS error code.
- Errors are returned, not thrown (consistent with Go error handling style).
- It executes the appropriate `git config` command to set the specified configuration (e.g., `git config remote.origin.pushurl <value>`).

## 3. Test Plan

- **Unit Test:** Test the function with valid inputs and verify that the correct `git config` command is constructed and executed.
- **Integration Test:** In a temporary Git repository, call the function to set a configuration value. Verify with `git config --get` that the value was correctly set.
- **Failure Case:** Call the function with an invalid remote name and verify it returns an error.
