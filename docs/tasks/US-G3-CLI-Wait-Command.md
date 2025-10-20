# Task: CLI `wait` Command

**Feature:** `US-G3 — Mirror Resilience & Read-After-Write`

## 1. Description

Implement the `git kv wait` command. This command allows a client (like a CI script) to pause execution until a specific commit is confirmed to be present on the GitHub mirror, providing a Read-Your-Writes (RYW) guarantee.

## 2. Acceptance Criteria

- A `git kv wait` command is added to the CLI.
- It accepts a required `--oid <hash>` flag specifying the ledger commit to wait for.
- It accepts an optional `--ns <namespace>` flag (defaults to `main` or the namespace defined in `.kv/policy.yaml`).
- It accepts an optional `--timeout <duration>` flag (e.g., `60s`, `5m`). **Default: 5m**.
- It accepts an optional `--poll-interval <duration>` flag (default `2s`). The command polls `git ls-remote origin refs/kv-mirror/<ns>` every `POLL_INTERVAL` (default 2 seconds, chosen to balance responsiveness and remote load).
- If a `git ls-remote` invocation fails, the command retries up to three additional attempts using exponential backoff delays of `1s`, `2s`, and `4s` between retries. Transient failures that succeed within these retries do not abort the wait.
- The command exits with status 0 as soon as the watermark's OID is the same as, or the target OID is an ancestor of the watermark OID (i.e., `git merge-base --is-ancestor <target> <watermark>` returns true).
- If the timeout is reached before the condition is met, the command exits with a non-zero status code and prints a clear message such as `ERROR: timeout waiting for refs/kv-mirror/<ns> to reach <target_oid>`.
- If all retry attempts for `git ls-remote` fail before reaching the timeout, the command exits non-zero and prints a clear error message (e.g., `failed to poll refs/kv-mirror/<ns> after 4 attempts: <error>`).

## 3. Test Plan

- **Integration Test (Success):**
  - Start `git kv wait --oid <target_oid> --ns main --timeout 30s` in a background process.
  - 2 seconds after `wait` starts, update `refs/kv-mirror/main` to `<target_oid>` in a separate process.
  - Assert `wait` exits successfully (code 0) within 5 seconds.
- **Integration Test (Timeout):**
  - Run `git kv wait --oid <non_existent_oid> --ns main --timeout 5s`.
  - Assert the command exits with a timeout error (non-zero code) after exactly 5 seconds.
- **Integration Test (Network Failure - Transient):**
  - Configure a mock remote to simulate transient network failures for `git ls-remote` (e.g., fail twice, then succeed on the third attempt).
  - Run `git kv wait --oid <target_oid> --ns main`. Assert the command performs the backoff sequence (`1s`, `2s`) and then succeeds without exiting early.
- **Integration Test (Network Failure - Permanent):**
  - Configure a mock remote to simulate permanent network failures for `git ls-remote`.
  - Run `git kv wait --oid <target_oid> --ns main`. Assert the command makes exactly four total attempts (initial + three retries), emits the documented error message, and exits non-zero before the overall timeout.
- **Unit Test (Polling Logic):** Mock `git ls-remote` outputs (e.g., "not found" -> "not found" -> "found after N polls"). Verify the polling logic correctly determines success or timeout.
- **Unit Test (Ancestor Checking):** Test `git merge-base --is-ancestor` logic with mock OIDs:
  - Target is ancestor of current.
  - Target is not ancestor of current.
  - Unrelated branches.
  - Non-existent OIDs (expect timeout, not crash).
