# Task: Stargate Mirror Worker

**Feature:** `US-G3 — Mirror Resilience & Read-After-Write`

## 1. Description

Implement the background worker process on the Stargate server. This worker will read jobs from the mirror journal, push the specified refs to the GitHub remote, and handle retries and failures.

## 2. Acceptance Criteria

- The worker runs as a separate, long-running process or is triggered periodically.
- It reads new entries from the mirror journal file.
- For each entry, it executes `git push --atomic <mirror_remote> <refspec>...`.
- The `mirror_remote` is the pre-configured name for the GitHub remote (e.g., `github-origin`).
- The push includes the main data ref and its corresponding watermark ref (`refs/kv-mirror/<ns>`).
- If the push fails, the worker implements an exponential backoff retry mechanism with:
  - Initial delay: 500ms.
  - Maximum delay cap: 60s.
  - Maximum retry attempts: 10.
  - Randomized jitter: Yes (e.g., +/- 25% of delay).
- Successfully processed journal entries are **deleted** from the journal file.
- Failed journal entries (after exhausting retries) are moved to a separate "dead-letter" queue or logged as permanent failures.

## 3. Test Plan

- **Integration Test (Success):** Add an entry to the journal file. Run the worker and verify it performs the correct `git push` command to a mock GitHub remote and that the journal entry is marked as processed.
- **Integration Test (Failure):** Configure the mock GitHub remote to reject pushes. Run the worker and verify that it attempts to push and then correctly retries according to its backoff strategy.
- **Robustness:** Test that the worker can process multiple journal entries in sequence.
