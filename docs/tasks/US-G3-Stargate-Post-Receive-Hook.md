# Task: Stargate `post-receive` Hook for Mirroring

**Feature:** `US-G3 — Mirror Resilience & Read-After-Write`

## 1. Description

Implement or update the `post-receive` hook on the Stargate server. After a push is successfully accepted and validated by `pre-receive`, this hook will be responsible for queueing a job to mirror the changes to the GitHub remote.

## 2. Acceptance Criteria

- The `post-receive` hook is an executable script in the Stargate repo's `hooks` directory.
- The hook reads the `old`, `new`, and `ref` values from stdin for all updated refs.
- For each relevant ref update (e.g., `refs/kv/*`, `refs/kv-index/*`), it writes a structured log entry to a journal file (e.g., `.stargate/mirror-journal.ndjson`).
- The log entry should contain all information needed for a background worker to process the push, such as the ref name and the new OID.
- The hook must be robust and not fail silently.

## 3. Test Plan

- **Integration Test:** Perform a successful push to a test Stargate repository. After the push, verify that the `mirror-journal.ndjson` file exists and contains a correctly formatted entry corresponding to the push.
- **Edge Case:** Push multiple refs at once and verify that a log entry is created for each updated ref.
- **Edge Case:** Test that pushes to non-`git-kv` branches do not create entries in the mirror journal.
