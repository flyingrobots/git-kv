# Task: Client-Side CAS Error Handling

**Feature:** `US-4 — CAS on branch`

## 1. Description

Implement the client-side logic to handle the specific rejection that occurs during a failed Compare-And-Swap (CAS) operation. The user should receive a clear, understandable error instead of a generic `git push` failure.

## 2. Acceptance Criteria

- When a `git push` from a `set` or `mset` command with `--expect-tree` is rejected by the Stargate, the client inspects the stderr output from Git.
- The client looks for the specific, machine-readable error message sent by the Stargate for a CAS failure.
- If this message is detected, the client CLI exits with a specific exit code (e.g., `2` as per the spec).
- The CLI prints a user-friendly error message, such as "Error: A concurrent modification occurred. Please try again."

## 3. Test Plan

- **Integration Test:** Use the CAS failure integration test from the Stargate task. Run the client command that is expected to fail and assert that it exits with the correct exit code (e.g., `2`) and that the error message printed to the user is the friendly, expected message.
- **Unit Test:** Write a unit test that feeds mock `git` stderr output (both for CAS failure and other generic failures) to the error handling function and verify it behaves correctly.
