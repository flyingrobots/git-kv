# Task: Client-Side CAS Error Handling

**Feature:** `US-4 — CAS on branch`

## 1. Description

Implement the client-side logic to handle the specific rejection that occurs during a failed Compare-And-Swap (CAS) operation. The user should receive a clear, understandable error instead of a generic `git push` failure.

## 2. Acceptance Criteria

- **Dependency:** This task depends on `US-4-Stargate-CAS-Check.md` for the exact CAS conflict error format.
- When a `git push` from a `set` or `mset` command with `--expect-tree` is rejected by the Stargate, the client splits Git stderr into lines, trims leading/trailing whitespace on each line, and performs a case-sensitive equality check against `ERROR: CAS_CONFLICT: Expected tree OID mismatch.`
- If any trimmed line equals that exact string, the client CLI **must** exit with code `2` and print `Error: A concurrent modification occurred. Please try again.`
- If Git exits with code `2` but no trimmed line matches exactly, treat it as a generic Git failure: exit with code `1` and print `Error: Git operation failed. See git output for details.`
- If Git exits with code `0` or `1` and a trimmed line matches the CAS string, do not override Git's outcome; exit with the original Git exit code and surface Git stderr unchanged.
- If Git exits with code `2` and stderr contains a partial or garbled CAS-like line (substring match but not exact), treat it as ambiguous: exit with code `1` and print `Error: Git operation failed. See git output for details.`

## 3. Test Plan

Helper function signature: `parseCasError(stderr: string): CASErrorResult`, where `CASErrorResult` has fields `{ type: CASError; message: string; exitCode: number }` and `CASError` is an enum with at least `CAS_CONFLICT`, `GENERIC_GIT_FAILURE`, and `PASS_THROUGH` variants.

- **Integration Test (CAS Failure):**
  - **Setup:** Replace the Stargate interaction with a local stub (e.g., inject a fake Stargate client or start an HTTP mock server) that always returns the CAS rejection payload (`ERROR: CAS_CONFLICT: Expected tree OID mismatch.`) and exit code `2` when the client performs the push.
  - **Action:** Run the client command against the stubbed Stargate to simulate the CAS failure deterministically.
  - **Assert:**
    - Exit code is `2` (exactly).
    - Stderr contains: "Error: A concurrent modification occurred. Please try again."
    - No exception/panic occurs.
- **Integration Test (Generic Git error):**
  - Mock git to exit with code `2` and stderr lacking the CAS line; assert the client exits `1` and prints `Error: Git operation failed. See git output for details.`
- **Integration Test (CAS string with success git exit):**
  - Mock git to exit `0` while emitting the CAS string; assert the client exits `0` and propagates git stderr without substituting the CAS message.
- **Integration Test (Partial CAS string):**
  - Mock git to exit `2` with stderr containing `ERROR: CAS_CONFL` (truncated); assert client exits `1` with generic git error message.
- **Unit Test (Error Message Parsing):**
  - **Setup:** Mock `git` stderr with the Stargate CAS failure message: "ERROR: CAS_CONFLICT: Expected tree OID mismatch."
  - **Action:** Call `const result = parseCasError(stderr: string): CASErrorResult`.
  - **Assert:**
    - `result.type` equals `CASError.CAS_CONFLICT`.
    - `result.message` equals exactly `Error: A concurrent modification occurred. Please try again.`
    - `result.exitCode` equals `2`.
- **Unit Test (No match fallback):** Provide stderr without the CAS line and git exit `2`; assert `parseCasError` returns `CASErrorResult` with `type = CASError.GENERIC_GIT_FAILURE`, message `Error: Git operation failed. See git output for details.`, and exit code `1`.
- **Unit Test (Partial match rejected):** Provide stderr containing `ERROR: CAS_CONFLICT` (missing rest) and git exit `2`; assert the parser returns `CASError.GENERIC_GIT_FAILURE` with the generic message and exit code `1`.
- **Unit Test (CAS string with git success):** Provide CAS line but git exit `0`; assert the parser returns `CASErrorResult` with `type = CASError.PASS_THROUGH`, preserves the original exit code `0`, and `result.message` echoes the original git stderr (no substitution).
