# Task: Stargate Mirror Watermark

**Feature:** `US-G3 — Mirror Resilience & Read-After-Write`

## 1. Description

As part of the mirror worker's logic, implement the crucial step of updating a watermark ref (`refs/kv-mirror/<ns>`) on the GitHub remote. This ref provides a reliable, client-visible signal of the current mirror state and is essential for the `git kv wait` command.

## 2. Acceptance Criteria

- After the mirror worker successfully pushes a ledger ref (e.g., `refs/kv/main`) to the GitHub remote, it performs a second push.
- This second push updates the `refs/kv-mirror/main` ref on the GitHub remote to point to the exact same commit OID as the `refs/kv/main` ref that was just pushed.
- This update should be done atomically with the main push if possible, or as a separate, final step.

## 3. Test Plan

- **Integration Test:** After the mirror worker test successfully pushes a change, add an assertion to verify that the `refs/kv-mirror/main` ref on the mock GitHub remote has been updated to the correct commit OID.
- **Edge Case:** Test a scenario where the main push succeeds but the watermark push fails. The mirror worker should detect this and retry the watermark push.
