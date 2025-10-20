# Task: Client-Side CAS Error Handling

**Feature:** `US-4 — CAS on branch`

## 1. Description

Implement the client-side logic to handle the specific rejection that occurs during a failed Compare-And-Swap (CAS) operation. The user should receive a clear, understandable error instead of a generic `git push` failure.

## 2. Acceptance Criteria

- **Dependency:** This task depends on `US-4-Stargate-CAS-Check.md` for the exact CAS conflict error format.
- When a `git push` from a `set` or `mset` command with `--expect-tree` is rejected by the Stargate, the client inspects the stderr output from Git.
- The client must detect the exact CAS conflict error message: "ERROR: CAS_CONFLICT: Expected tree OID mismatch."
- The client CLI **must** exit with exit code `2`.
- The client CLI **must** print the exact user-facing message: "Error: A concurrent modification occurred. Please try again."

## 3. Test Plan

- **Integration Test (CAS Failure):**
  - **Setup:** Execute the Stargate CAS Check integration test (`US-4-Stargate-CAS-Check.md`) to trigger a CAS failure.
  - **Action:** Run the client command that triggered the CAS failure.
  - **Assert:**
    - Exit code is `2` (exactly).
    - Stderr contains: "Error: A concurrent modification occurred. Please try again."
    - No exception/panic occurs.
- **Unit Test (Error Message Parsing):**
  - **Setup:** Mock `git` stderr with the Stargate CAS failure message: "ERROR: CAS_CONFLICT: Expected tree OID mismatch."
  - **Action:** Call the error-handling function responsible for parsing `git` stderr.
  - **Assert:**
    - Returns an error type indicating CAS failure.
    - The suggested user-facing message is "Error: A concurrent modification occurred. Please try again."
