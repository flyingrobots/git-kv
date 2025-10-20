# Task: CLI `wait` Command

**Feature:** `US-G3 — Mirror Resilience & Read-After-Write`

## 1. Description

Implement the `git kv wait` command. This command allows a client (like a CI script) to pause execution until a specific commit is confirmed to be present on the GitHub mirror, providing a Read-Your-Writes (RYW) guarantee.

## 2. Acceptance Criteria

- A `git kv wait` command is added to the CLI.
- It accepts a required `--oid <hash>` flag specifying the ledger commit to wait for.
- It accepts an optional `--timeout <duration>` flag (e.g., `60s`, `5m`). Default should be reasonable (e.g., 5 minutes).
- The command periodically runs `git ls-remote origin refs/kv-mirror/<ns>` to check the OID of the watermark.
- The command exits with success (0) as soon as the watermark's OID is the same as, or a descendant of, the OID specified in the `--oid` flag.
- If the timeout is reached before the condition is met, the command exits with a non-zero status code and an error message.

## 3. Test Plan

- **Integration Test (Success):**
  - Start `git kv wait --oid <target_oid> --timeout 30s` in a background process.
  - 2 seconds after `wait` starts, update `refs/kv-mirror/main` to `<target_oid>` in a separate process.
  - Assert `wait` exits successfully (code 0) within 5 seconds.
- **Integration Test (Timeout):**
  - Run `git kv wait --oid <non_existent_oid> --timeout 5s`.
  - Assert the command exits with a timeout error (non-zero code) after exactly 5 seconds.
- **Integration Test (Network Failure - Transient):**
  - Configure a mock remote to simulate transient network failures for `git ls-remote` (e.g., fail 2 times, then succeed).
  - Run `git kv wait`. Assert the command retries and eventually succeeds.
- **Integration Test (Network Failure - Permanent):**
  - Configure a mock remote to simulate permanent network failures for `git ls-remote`.
  - Run `git kv wait`. Assert the command exits with an immediate error or after a configured number of retries.
- **Unit Test (Polling Logic):** Mock `git ls-remote` outputs (e.g., "not found" -> "not found" -> "found after N polls"). Verify the polling logic correctly determines success or timeout.
- **Unit Test (Ancestor Checking):** Test `git merge-base --is-ancestor` logic with mock OIDs:
  - Target is ancestor of current.
  - Target is not ancestor of current.
  - Unrelated branches.
  - Non-existent OIDs (expect timeout, not crash).
