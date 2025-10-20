# Task: Epoch-Aware Commit Generation

**Feature:** `US-6 — Epoch Rollover`

## 1. Description

Modify the client-side commit generation logic for `set` and `mset` to be epoch-aware. All new data commits must be tagged with the current epoch, and the first commit of a new epoch must link to the previous one.

## 2. Acceptance Criteria

- Before generating a commit, the client logic reads the current epoch name from the `refs/kv-epoch/<ns>` ref. This read **must** be performed immediately before the push operation to minimize staleness.
- Every new ledger commit created by `git-kv` **must** include an ASCII trailer line exactly formatted as `Epoch: <epoch_name>\n`. `<epoch_name>` must be RFC-1123 token safe; any other characters must be percent-encoded prior to inclusion.
- When creating the very first commit in a new epoch, the commit **must** additionally include `Parent-Epoch: <previous_epoch_name>\n` using the same encoding rules.
- Trailers must comply with Git trailer semantics (no duplicate `Epoch` or `Parent-Epoch` lines; duplicates are errors).
- Chunk blobs for the commit must be written exclusively under `refs/kv-chunks/<ns>@<epoch_name>` (exact format, no aliases). Epoch chunk refs are immutable, retained for 90 days, and garbage-collected by a daily background job after an archival snapshot is written for audit.
- **Concurrency Handling:** If the epoch changes between read and push, Stargate `pre-receive` must reject the commit when the `Epoch:` trailer differs from the authoritative epoch. The hook returns Git exit code `1` with stderr `REJECTED: EPOCH_MISMATCH current=<current_epoch> expected=<provided_epoch>` and HTTP clients receive `409 Conflict` with body `{"error":"epoch_mismatch","current":"<current_epoch>","provided":"<provided_epoch>"}`.
- Clients encountering `epoch_mismatch` **must** implement exponential backoff retry: initial delay 200 ms, multiplier 2, maximum delay 3200 ms, up to 6 attempts. After exhausting retries, clients surface a categorized `EpochMismatchError` to callers without further retries.

## 3. Test Plan

- **Integration Test:** After an epoch rollover, create a new key with `git kv set`. Fetch the created commit object and inspect its message to verify it contains the correct `Epoch` trailer.
- **Integration Test:** Inspect the first commit of the new epoch and verify it contains the correct `Parent-Epoch` trailer.
- **Unit Test:** Write a unit test for the commit generation function that mocks the current epoch and verifies the correct trailers are added to the commit message.
- **Integration Test (Epoch Rollover Mid-Operation):**
  1. Start a `git kv mset` operation on the client.
  2. While the client is processing (e.g., after tree construction but before push), trigger an `git kv epoch new` on the Stargate.
  3. Assert that the client's push is rejected by the Stargate due to an epoch mismatch.
  4. Assert that the client surfaces `REJECTED: EPOCH_MISMATCH current=<...> expected=<...>` on stderr and HTTP 409 JSON in API mode.
  5. Assert that the client retries with exponential backoff (capturing retry intervals) and eventually succeeds or exits with categorized error after exhausting attempts.
