# Task: Client Git Configurator

**Feature:** `US-G12 — Discovery on Fresh Clone`

## 1. Description

Implement the client-side logic responsible for modifying the local Git configuration. This module will be used by `git kv remote setup` to set the `origin.pushurl`.

## 2. Acceptance Criteria

- A function `ConfigureRemote(remoteName string, key string, value string) error` is defined.
- Inputs must be validated before execution: `remoteName` limited to `[A-Za-z0-9_.-]{1,64}`, `key` must begin with `remote.` and contain only `[A-Za-z0-9_.-]`, and `value` must be ≤ 2048 characters. `value` must parse as a valid Git URL (ssh:// or https://). Reject invalid inputs with `ConfigError{Code: INVALID_INPUT, ExitCode: 3}` or `INVALID_REMOTE` when URL parsing fails.
- Before writing configuration, call `git config --get remote.<remoteName>.url`; if the remote does not exist, return `REMOTE_NOT_FOUND` without making changes.
- The function must invoke Git via a safe command API that passes arguments as an array (e.g., `exec.Command("git", "config", key, value)`), never via shell interpolation. If a shell fallback is required, all arguments must be properly escaped; unsafe values should be rejected.
- Platform note: always call `exec.Command` (or equivalent) with argument arrays (`exec.Command("git", "config", key, value)`) and **never** invoke a shell (`sh -c`, `cmd /C`). On Unix-like systems this executes git directly; on Windows ensure no command string is constructed so `cmd.exe` is not invoked implicitly. If a shell fallback is ever considered, reject or strictly validate/escape inputs first. Test on Windows to confirm no shell interpretation occurs.
- Values containing spaces or special characters must be percent-encoded by the configurator prior to invoking Git; non-printable or UTF-8-invalid bytes must be rejected with `INVALID_INPUT`. Tests must cover encoded and rejected cases.
- The function returns `nil` on success.
-- On failure it returns a `ConfigError` with fields:
  - `Code`: one of `INVALID_INPUT`, `INVALID_REMOTE`, `REMOTE_NOT_FOUND`, `PERMISSION_DENIED`, `AUTH_FAILED`, `TIMEOUT`, `GIT_FAILED`, `GIT_INVALID_INVOCATION`, `GIT_FATAL`.
  - `Message`: human-readable explanation.
  - `ExitCode`: numeric exit code to propagate (see CLI spec).
  - `Details`: optional `ErrorDetails` struct with additional context (e.g., underlying git stderr or command arguments).
- Error mapping rules:
  - `INVALID_INPUT`: parameter validation failed.
  - `INVALID_REMOTE`: remoteName missing/empty, malformed, or value not parseable as URL.
  - `REMOTE_NOT_FOUND`: `git config --get remote.<name>.url` returns not found.
  - `PERMISSION_DENIED`: OS-level permission errors writing config.
  - `AUTH_FAILED`: authentication/SSH errors while validating remote URLs.
  - `TIMEOUT`: git command exceeded configured timeout.
  - `GIT_FAILED`: non-zero git exits not matched by other rules; include git exit code in `Details`.
  - `GIT_INVALID_INVOCATION`: git exits with code `2` (misuse/invalid invocation).
  - `GIT_FATAL`: git exits with code ≥128 (fatal error or signal) or when stderr indicates fatal hardware/permission issues.
- Git exit code mapping decision tree:
  1. If exit code == 0 → success (no error).
  2. If exit code == 1 and stderr contains "No such remote"/"not found" → `REMOTE_NOT_FOUND`; otherwise → `GIT_FAILED`.
  3. If exit code == 2 → `GIT_INVALID_INVOCATION`.
  4. If exit code >= 128 → `GIT_FATAL` with `Details.GitExitCode` set.
  5. Otherwise → `GIT_FAILED` with `Details.GitExitCode` set.
  - Examples:
    - git returns 1 with stderr `fatal: No such remote 'origin2'` → `REMOTE_NOT_FOUND`.
    - git returns 2 for bad flag → `GIT_INVALID_INVOCATION`.
    - git returns 128 after SIGTERM → `GIT_FATAL`.
- Errors are returned, not panicked, matching Go style.
- Git commands must be executed with context-aware timeouts (default 10s) and cancellation propagation.

## 3. Test Plan

- **Unit Test:** Test that valid inputs result in `exec.Command("git", "config", key, value)` style invocation (argument array) without shell usage.
- **Integration Test:** In a temporary Git repository, call the function to set a configuration value. Verify with `git config --get` that the value was correctly set.
- **Failure Case (Invalid Input):** Provide invalid remote names/values and assert `ConfigError` with `Code: INVALID_INPUT` and exit code 3.
- **Failure Case (Invalid URL):** Supply non-parsable push URL and expect `INVALID_REMOTE` error.
- **Failure Case (Injection Attempt):** Provide values containing shell metacharacters; assert rejection with `INVALID_INPUT` and confirm command array invocation (no shell).
- **Failure Case (Remote Missing):** Use a repo without the remote; expect `REMOTE_NOT_FOUND`.
- **Failure Case (Git Error):** Simulate git returning non-zero; expect `GIT_FAILED` with captured exit code.
- **Failure Case (Exit Mapping):** Simulate git exit `1` with "No such remote" → assert `REMOTE_NOT_FOUND`; exit `2` → `GIT_INVALID_INVOCATION`; exit `128` → `GIT_FATAL`.
- **Timeout/AUTH Tests:** Simulate timeout and auth errors; assert appropriate codes.
- **Percent-Encoding Test:** Provide value with spaces; ensure configurator percent-encodes and git receives encoded value.
- `ConfigError` is defined as:
  ```go
  type ErrorCode string

  type ConfigError struct {
      Code     ErrorCode     `json:"code"`        // e.g., "invalid_input"
      Message  string        `json:"message"`
      ExitCode int           `json:"exit_code"`
      Details  *ErrorDetails `json:"details,omitempty"`
  }

  type ErrorDetails struct {
      RemoteName   string   `json:"remote_name,omitempty"`
      GitExitCode  int      `json:"git_exit_code,omitempty"`
      CommandArgs  []string `json:"command_args,omitempty"`
      Cause        string   `json:"cause,omitempty"`
  }
  ```
  - JSON field names use snake_case; Go field names remain exported CamelCase to comply with Go style.
- `ConfigError.ExitCode` values must align with `git kv remote setup` CLI exit codes (e.g., INVALID_INPUT -> 3, REMOTE_NOT_FOUND -> 1, PERMISSION_DENIED -> 2, AUTH_FAILED -> 4, TIMEOUT -> 5, GIT_FAILED / GIT_INVALID_INVOCATION / GIT_FATAL -> 5, while still including the raw git exit code in `Details`).
