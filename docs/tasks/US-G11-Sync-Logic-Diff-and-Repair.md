# Task: Sync Logic: Diff and Repair

**Feature:** `US-G11 — Split-Brain Recovery`

## 1. Description

Implement the core logic for `git kv stargate sync` to detect divergences between the Stargate and its GitHub mirror, and to perform a safe repair (fast-forward push).

## 2. Acceptance Criteria

- The logic compares the OIDs of all refs matching the glob patterns `refs/kv/*`, `refs/kv-index/*`, `refs/kv-chunks/*`, `refs/kv-epoch/*`, `refs/kv-archive/*`, `refs/kv-watchlog/*`, and `refs/kv-mirror/*` on the Stargate with their counterparts on the GitHub mirror.
- **Error Reporting:** All errors (e.g., network issues, non-fast-forward rejections) are written to `stderr` and a structured log file (e.g., `stargate-sync.log`). Errors **must** include ref name, source OID, target OID, and a human-readable reason.
- **`--dry-run` mode:**
  - Identifies refs that are ahead on Stargate (mirror is behind).
  - Identifies refs that are ahead on the mirror (Stargate is behind, indicating divergence).
  - Prints a report of these differences without making any changes.
  - Exit code: `0`.
- **`--repair` mode:**
  - For refs where Stargate is ahead, it attempts a `git push <mirror_remote> <refspec>` (fast-forward push).
  - If the mirror is ahead (non-fast-forward), it reports an error (as defined in Error Reporting) and does not push that specific ref.
  - **Retry/Abort Policy:** The command continues processing remaining refs on individual failures and aggregates/reports all failures at the end.
  - Exit code: `0` (all good/no repairs), `1` (non-fatal repair failures but completed processing), `2` (fatal error that stops processing).

## 3. Test Plan

- **Integration Test (Dry Run):** Simulate a scenario where the Stargate is ahead of the mirror. Run `sync --dry-run` and verify it correctly reports the differences.
- **Integration Test (Repair Success):** Simulate a scenario where the Stargate is ahead. Run `sync --repair` and verify the mirror is updated.
- **Integration Test (Repair Failure):** Simulate a scenario where the mirror is ahead (diverged). Run `sync --repair` and verify it reports a non-fast-forward error and does not push.
