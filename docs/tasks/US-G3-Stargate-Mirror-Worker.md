# Task: Stargate Mirror Worker

**Feature:** `US-G3 — Mirror Resilience & Read-After-Write`

## 1. Description

Implement the background worker process on the Stargate server. This worker will read jobs from the mirror journal, push the specified refs to the GitHub remote, and handle retries and failures.

## 2. Acceptance Criteria

- The worker runs as a single-threaded, long-running process that processes journal entries sequentially to preserve ordering. Concurrency may be increased in the future but is out of scope for this task.
- It tails the mirror journal file, respecting the append-only semantics defined in the post-receive hook, and replays entries starting at the last durable checkpoint.
- Progress is tracked via an atomic checkpoint file (e.g., `.stargate/mirror-worker.offset`). After successfully processing an entry, the worker fsyncs the checkpoint to persist the new offset. On startup, the worker resumes from the checkpoint.
- Each journal entry must be processed idempotently. The worker must detect if an entry has already been mirrored (e.g., by comparing remote OIDs) and skip re-pushing to avoid duplicates.
- Upon successful processing, the worker must write a status record (e.g., appended to `.stargate/mirror-status.ndjson`) marking the `entry_id` as `"completed"`; dead-lettered entries are recorded with status `"dead-letter"`. Status writes must be atomic and fsynced similar to checkpoints.
- For each entry, the worker runs `git push --atomic <mirror_remote> <refspec>…` (data ref plus `refs/kv-mirror/<ns>`) with a configurable timeout (`PUSH_TIMEOUT`, default 30s). If the command times out or fails, retry up to 2 additional attempts with exponential backoff (initial 500ms, multiplier 2x, capped at 10s) before marking the entry failed.
- The `mirror_remote` is the configured GitHub remote name (e.g., `github-origin`).
- Malformed or corrupted journal entries (invalid JSON, truncated data, missing required fields) must be detected, retried up to 3 times with backoff (initial 100ms, max 1s). After retries are exhausted, the entry is moved to a durable dead-letter queue with original payload, error detail, timestamps, and metrics incremented; processing then continues with the next entry.
- Any entry whose push ultimately fails after retries (including timeouts, auth errors, protected refs) must also be moved to the dead-letter queue with full context. Logging alone is insufficient.
- Dead-letter queue must be queryable (e.g., `.stargate/mirror-dead-letter.ndjson`), retain entries for at least 30 days, and support manual requeue by operators (documented procedure).
- Checkpoint and dead-letter writes must be atomic (write temp + rename + fsync) to survive crashes.

## 3. Crash Recovery & Idempotency

- On startup after a crash, the worker replays from the last committed checkpoint. Because operations are idempotent, reprocessing an entry must be safe.
- If the worker crashes mid-entry (before checkpoint update), the entry remains unacknowledged and will be retried on restart.
- Metrics and logs must surface counts of processed, retried, and dead-lettered entries.

## 4. Test Plan

- **Integration Test (Success):** Add an entry to the journal file. Run the worker and verify it performs the correct `git push` command to a mock GitHub remote and that the journal entry is marked as processed.
- **Integration Test (Status Record):** After a successful push, verify that `.stargate/mirror-status.ndjson` contains a `completed` record for the processed `entry_id`.
- **Integration Test (Failure with Retries):** Configure the mock GitHub remote to reject pushes temporarily. Run the worker and verify retries follow the documented backoff and succeed, updating the checkpoint once done.
- **Integration Test (Dead-Letter on Persistent Failure):** Force the remote to reject pushes permanently. Verify the worker moves the entry to the dead-letter queue after retries and logs the failure.
- **Integration Test (Malformed Entry):** Inject invalid JSON into the journal; ensure the worker retries the configured number of times, records the entry in dead-letter, and continues processing subsequent valid entries.
- **Crash Recovery Test:** Process several entries, crash the worker mid-run, restart, and ensure it resumes from the last checkpoint without duplicating successfully mirrored entries.
- **Robustness:** Test processing of multiple entries sequentially, ensuring the checkpoint advances and no entries are skipped.
