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

- **Integration Test (Success):** In a test repo, run `wait` for an OID. In a separate process, update the `refs/kv-mirror/main` ref to that OID. Verify the `wait` command exits successfully.
- **Integration Test (Timeout):** Run `wait` for an OID that will never be set. Verify the command times out after the specified duration and returns an error.
- **Unit Test:** Test the polling logic in isolation.
- **Unit Test:** Test the ancestor-checking logic (`git merge-base --is-ancestor`).
