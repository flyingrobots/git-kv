# Task: Stargate Mirror Watermark

**Feature:** `US-G3 — Mirror Resilience & Read-After-Write`

## 1. Context

- **Mirror worker:** background process that consumes the mirror journal and mirrors successful ledger pushes to the GitHub remote.
- **GitHub remote:** the external Git hosting provider that clients read from (typically `origin`).
- **`git kv wait`:** CLI command that polls the mirror watermark to guarantee read-after-write visibility; depends on accurate `refs/kv-mirror/<ns>` updates.

## 2. Description

As part of the mirror worker's logic, implement the crucial step of updating a watermark ref (`refs/kv-mirror/<ns>`) on the GitHub remote. This ref provides a reliable, client-visible signal of the current mirror state and is essential for the `git kv wait` command.

## 3. Acceptance Criteria

- The mirror worker **must** perform a single `git push --atomic` command that includes both the ledger ref (e.g., `refs/kv/main`) and its corresponding watermark ref (`refs/kv-mirror/main`).
- This atomic push updates `refs/kv-mirror/main` on the GitHub remote to point to the same commit OID as the `refs/kv/main` ref that was just pushed.
- If the atomic push fails, neither `refs/kv/main` nor `refs/kv-mirror/main` may be updated on the remote; the worker must treat the entry as unprocessed and schedule a retry.
- Retry policy: attempt the atomic push up to 3 total times (initial attempt plus 2 retries) using exponential backoff with an initial delay of 500ms, a 2x multiplier per retry, and a cap of 10s. Apply the backoff between attempts and surface metrics/logs for each retry.

## 4. Test Plan

- **Integration Test:** After the mirror worker test successfully pushes a change, add an assertion to verify that the `refs/kv-mirror/main` ref on the mock GitHub remote has been updated to the correct commit OID.
- **Edge Case (Watermark Push Failure):**
  - Simulate a scenario where the `git push --atomic` (containing both ledger and watermark refs) fails due to a transient error on the watermark ref.
  - Assert that the mirror worker retries according to the 3-attempt policy (delays approx. 500ms then 1s, capped at 10s) and stops after the third failure.
  - Assert that after exhausting retries, an error is logged, the entry remains unprocessed, and the next worker cycle reattempts the push.
