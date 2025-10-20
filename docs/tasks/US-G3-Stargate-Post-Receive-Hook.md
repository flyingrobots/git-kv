# Task: Stargate `post-receive` Hook for Mirroring

**Feature:** `US-G3 — Mirror Resilience & Read-After-Write`

## 1. Description

Implement or update the `post-receive` hook on the Stargate server. After a push is successfully accepted and validated by `pre-receive`, this hook will be responsible for queueing a job to mirror the changes to the GitHub remote.

## 2. Acceptance Criteria

- The `post-receive` hook is an executable script in the Stargate repo's `hooks` directory.
- The hook reads the `old`, `new`, and `ref` values from stdin for all updated refs.
- For each relevant ref update (e.g., `refs/kv/*`, `refs/kv-index/*`), it writes a single-line NDJSON entry to the journal file (e.g., `.stargate/mirror-journal.ndjson`).
- The NDJSON schema for each entry **must** be:
  ```json
  {
    "timestamp": "ISO8601 string",
    "event": "string (e.g., ref_update)",
    "ref": "string (e.g., refs/kv/main)",
    "old_oid": "hex string | null",
    "new_oid": "hex string",
    "pusher": "string | null (e.g., user@example.com)",
    "repo": "string | null (e.g., flyingrobots/git-kv)",
    "source": "string (e.g., post-receive)",
    "meta": "object | null (additional metadata)"
  }
  ```
- **Example NDJSON Entry:**
  ```json
  {"timestamp":"2025-10-26T12:34:56Z","event":"ref_update","ref":"refs/kv/main","old_oid":"a1b2c3d4...","new_oid":"e5f6g7h8...","pusher":"user@example.com","repo":"flyingrobots/git-kv","source":"post-receive","meta":{"txn_id":"01JBAX..."}}
  ```
- Each entry **must** be atomically appended (single-line JSON) to the journal file.
- **Robustness & Logging:**
  - The hook logs all errors to a dedicated log file (e.g., `stargate-post-receive.log`) with timestamps and details.
  - If the journal write fails, the hook exits with a non-zero status and reports the error to `git` (which is then shown to the client).
  - The hook handles partial ref updates gracefully; if some refs succeed and others fail, it logs the failures but continues processing other refs.

## 3. Test Plan

- **Integration Test:** Perform a successful push to a test Stargate repository. After the push, verify that the `mirror-journal.ndjson` file exists and contains a correctly formatted entry corresponding to the push.
- **Edge Case:** Push multiple refs at once and verify that a log entry is created for each updated ref.
- **Edge Case:** Test that pushes to non-`git-kv` branches do not create entries in the mirror journal.
