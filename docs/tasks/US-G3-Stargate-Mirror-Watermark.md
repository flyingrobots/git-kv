# Task: Stargate Mirror Watermark

**Feature:** `US-G3 — Mirror Resilience & Read-After-Write`

## 1. Description

As part of the mirror worker's logic, implement the crucial step of updating a watermark ref (`refs/kv-mirror/<ns>`) on the GitHub remote. This ref provides a reliable, client-visible signal of the current mirror state and is essential for the `git kv wait` command.

## 2. Acceptance Criteria

- The mirror worker **must** perform a single `git push --atomic` command that includes both the ledger ref (e.g., `refs/kv/main`) and its corresponding watermark ref (`refs/kv-mirror/main`).
- This atomic push updates `refs/kv-mirror/main` on the GitHub remote to point to the same commit OID as the `refs/kv/main` ref that was just pushed.

## 3. Test Plan

- **Integration Test:** After the mirror worker test successfully pushes a change, add an assertion to verify that the `refs/kv-mirror/main` ref on the mock GitHub remote has been updated to the correct commit OID.
- **Edge Case (Watermark Push Failure):**
  - Simulate a scenario where the `git push --atomic` (containing both ledger and watermark refs) fails due to a transient error on the watermark ref.
  - Assert that the mirror worker retries the push with exponential backoff (e.g., max 3 retries).
  - Assert that after exhausting retries, an error is logged, but the journal entry is **not** marked as processed, ensuring it will be retried on the next worker cycle.
