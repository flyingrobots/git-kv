# Task: Sync Logic: Diff and Repair

**Feature:** `US-G11 — Split-Brain Recovery`

## 1. Description

Implement the core logic for `git kv stargate sync` to detect divergences between the Stargate and its GitHub mirror, and to perform fast-forward repairs only when the Stargate ref is an ancestor of the mirror ref (or vice versa). Before pushing, the logic must verify ancestry, object reachability, and absence of unexpected mirror force-pushes by comparing fetched OIDs and reflog timestamps; repairs are skipped when these preconditions are not met.

## 2. Acceptance Criteria

- The logic compares the OIDs of all refs matching the glob patterns `refs/kv/*`, `refs/kv-index/*`, `refs/kv-chunks/*`, `refs/kv-epoch/*`, `refs/kv-archive/*`, `refs/kv-watchlog/*`, and `refs/kv-mirror/*` on the Stargate with their counterparts on the GitHub mirror.
- Refs that exist on only one side (orphaned or stale) must be detected and reported with suggested actions (create, delete, or investigate).
- **Error Reporting:** All errors (e.g., network issues, non-fast-forward rejections) are written to `stderr` and a structured log file (e.g., `stargate-sync.log`). Errors **must** include ref name, source OID, target OID, and a human-readable reason.
- **`--dry-run` mode:**
  - Identifies refs that are ahead on Stargate (mirror is behind).
  - Identifies refs that are ahead on the mirror (Stargate is behind, indicating divergence).
  - Prints a report of these differences without making any changes.
  - Exit code: `0`.
- **`--repair` mode:**
  - For refs where Stargate is ahead, it attempts a `git push <mirror_remote> <refspec>` (fast-forward push).
  - If the mirror is ahead (non-fast-forward), it reports an error (as defined in Error Reporting) and does not push that specific ref.
  - Prior to any push, confirm the mirror ref is an ancestor of the Stargate ref, all intermediate commits exist locally, and no mirror reflog entry newer than the fetch timestamp changed the head; otherwise mark the ref as unsafe and report.
  - **Retry/Abort Policy:** The command continues processing remaining refs on individual failures and aggregates/reports all failures at the end.
  - Exit code: `0` (all good/no repairs), `1` (non-fatal repair failures but completed processing), `2` (fatal error that stops processing).

## 3. Test Plan

- **Integration Test (Dry Run):** Simulate Stargate ahead of mirror; run `sync --dry-run` and verify differences reported without changes.
- **Integration Test (Repair Success):** Fast-forwardable case; verify ancestry check passes and mirror updates to Stargate head.
- **Integration Test (Mirror Ahead Divergence):** Mirror has extra commit; ensure `--repair` flags non-fast-forward and skips push.
- **Integration Test (Orphaned Refs):** Refs exist only on Stargate; confirm dry-run/repair reports missing mirror refs and optionally recreates when allowed.
- **Integration Test (Stale Mirror Ref Deletion):** Mirror has obsolete ref absent on Stargate; verify policy for deletion (report or remove) executes as specified.
- **Integration Test (Network Failure):** Introduce network drop during dry-run and mid-repair; assert appropriate exit codes (2 for fatal), partial state rollback, and logs.
- **Integration Test (Permission Denied):** Simulate push denied on specific ref; ensure command continues with others and aggregates failure.
- **Integration Test (Large Ref Set):** 100k refs with mixed states; measure runtime and ensure completion within configured timeout without excessive memory.
- **Integration Test (Concurrent Sync):** Launch two sync operations simultaneously; ensure locking prevents conflicts or second run exits gracefully.
- **Integration Test (Glob Collision):** Introduce refs like `refs/kv/foo` and `refs/kv-foo`; verify glob filtering handles them correctly.
- **Integration Test (Object Validation):** Remove an intermediate commit locally; ensure repair refuses to push and reports missing object.
- **Integration Test (Remote Force-Push Detection):** Change mirror after initial fetch; verify tool detects reflog mismatch and skips push.
