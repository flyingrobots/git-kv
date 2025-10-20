# Task: Sync Logic: Diff and Repair

**Feature:** `US-G11 — Split-Brain Recovery`

## 1. Description

Implement the core logic for `git kv stargate sync` to detect divergences between the Stargate and its GitHub mirror, and to perform a safe repair (fast-forward push).

## 2. Acceptance Criteria

- The logic compares the OIDs of all relevant refs (e.g., `refs/kv/*`, `refs/kv-index/*`) on the Stargate with their counterparts on the GitHub mirror.
- **`--dry-run` mode:**
  - Identifies refs that are ahead on Stargate (mirror is behind).
  - Identifies refs that are ahead on the mirror (Stargate is behind, indicating divergence).
  - Prints a report of these differences without making any changes.
- **`--repair` mode:**
  - For refs where Stargate is ahead, it attempts a `git push <mirror_remote> <refspec>` (fast-forward push).
  - If the mirror is ahead (non-fast-forward), it reports an error and does not push.

## 3. Test Plan

- **Integration Test (Dry Run):** Simulate a scenario where the Stargate is ahead of the mirror. Run `sync --dry-run` and verify it correctly reports the differences.
- **Integration Test (Repair Success):** Simulate a scenario where the Stargate is ahead. Run `sync --repair` and verify the mirror is updated.
- **Integration Test (Repair Failure):** Simulate a scenario where the mirror is ahead (diverged). Run `sync --repair` and verify it reports a non-fast-forward error and does not push.
