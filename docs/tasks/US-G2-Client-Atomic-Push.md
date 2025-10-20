# Task: Client-Side Atomic Push

**Feature:** `US-G2 — Atomic transaction via Stargate`

## 1. Description

Implement the final step of the `mset` command: pushing the newly created ledger and index commits to the Stargate remote. The push must be atomic to ensure that both refs are updated together or not at all.

## 2. Acceptance Criteria

- The client executes a `git push` command.
- The push command targets the `pushurl` of the `origin` remote (the Stargate).
- The push command includes the refspecs for both the new ledger commit and the new index commit.
- The push command uses the `--atomic` flag.
- If the push is successful, the `mset` command exits with code 0.
- If the push is rejected by the Stargate (for any reason, like a failed attestation), the client captures the error from Git and presents a user-friendly message before exiting with a non-zero code.

## 3. Test Plan

- **Integration Test:** In a test environment with a mock Stargate, execute a valid `mset` operation and verify the atomic push succeeds.
- **Integration Test (Rejection):** Configure the mock Stargate to reject pushes. Execute an `mset` operation and verify the command fails with a clear error message derived from the Git output.
- **Integration Test (Non-Atomic):** If possible, simulate a scenario where one ref could be pushed but the other couldn't, and verify that `--atomic` causes the entire push to fail.
