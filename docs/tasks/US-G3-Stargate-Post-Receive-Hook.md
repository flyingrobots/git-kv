# Task: Stargate `post-receive` Hook for Mirroring

**Feature:** `US-G3 — Mirror Resilience & Read-After-Write`

## 1. Description

Implement or update the `post-receive` hook on the Stargate server. After a push is successfully accepted and validated by `pre-receive`, this hook queues work by appending entries to `.stargate/mirror-journal.ndjson`, which the mirror worker consumes to mirror changes to the GitHub remote.

## 2. Acceptance Criteria

- The `post-receive` hook is an executable script in the Stargate repo's `hooks` directory.
- The hook reads the `old`, `new`, and `ref` values from stdin for all updated refs.
- For each ref matching `refs/kv/*` or `refs/kv-index/*`, it appends a single-line NDJSON entry to `.stargate/mirror-journal.ndjson` (the journal is the queue for mirror jobs).
- The hook must ensure `.stargate/` exists (creating with mode `0755` if necessary) and that `mirror-journal.ndjson` exists with mode `0644`; creation errors must be logged and cause a non-zero exit.
- The mirror worker defined in `US-G3-Stargate-Mirror-Worker` is the sole consumer of this journal; the hook must document this dependency in code comments or README updates.
- The NDJSON schema for each entry **must** be:

  ```json
  {
    "entry_id": "string (unique identifier, e.g., ULID)",
    "timestamp": "ISO8601 UTC string with trailing Z",
    "event": "string (e.g., ref_update)",
    "ref": "string (e.g., refs/kv/main)",
    "old_oid": "string | null (40- or 64-character hex SHA)",
    "new_oid": "string (40- or 64-character hex SHA)",
    "pusher": "string | null (e.g., user@example.com)",
    "repo": "string | null (e.g., flyingrobots/git-kv)",
    "source": "string (e.g., post-receive)",
    "status": "string (initially 'pending')",
    "meta": "object | null (additional metadata)"
  }
  ```
- **Example NDJSON Entry:**

  ```json
  {"entry_id":"01JBAXF5Z8J8M7R2TK7G","timestamp":"2025-10-26T12:34:56Z","event":"ref_update","ref":"refs/kv/main","old_oid":"a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4e5f6a1b2","new_oid":"0f1e2d3c4b5a0f1e2d3c4b5a0f1e2d3c4b5a0f1e","pusher":"user@example.com","repo":"flyingrobots/git-kv","source":"post-receive","status":"pending","meta":{"txn_id":"01JBAXF5Z8J8M7R2"}}
  ```
- `pusher` should be populated from environment variables provided by the Git server (e.g., `GIT_PUSHER_NAME`, `GIT_AUTHOR_EMAIL`, `SSH_AUTH_USER`); if unavailable, set to `null`.
- `repo` should be derived from `GIT_DIR` or a configured repository identifier; if unavailable, set to `null`.
- The `status` field must be initialised to `"pending"`; downstream mirror workers will update this field when processing entries.
- Each entry **must** be atomically appended by acquiring an exclusive file lock (e.g., `flock`), writing the JSON line + newline in one write, calling `fsync`, and releasing the lock. Windows deployments must use the platform equivalent. Lock contention must be retried with a short backoff.
- **Robustness & Logging:**
  - The hook logs all actions and errors to a dedicated log file (e.g., `stargate-post-receive.log`) with timestamps and details.
  - If any journal append fails, the hook exits with a non-zero status so the push is rejected and the pusher is informed.
  - When multiple refs are updated, the hook appends entries for successful refs, logs failures for unsuccessful ones, and exits non-zero to signal partial failure.

## 3. Test Plan

- **Integration Test:** Perform a successful push to a test Stargate repository. After the push, verify that `.stargate/mirror-journal.ndjson` exists and contains a correctly formatted entry corresponding to the push.
- **Edge Case:** Push multiple refs at once and verify that entries are created only for refs matching the allowed patterns (`refs/kv/*`, `refs/kv-index/*`).
- **Edge Case:** Push to refs outside those patterns and verify that no entries are created in the journal.
- **Failure Simulation:** Inject a permission or disk error for one ref update; assert the hook logs the failure, records the successful refs, and exits non-zero.
