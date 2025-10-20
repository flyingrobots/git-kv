# Task: Epoch-Aware Commit Generation

**Feature:** `US-6 — Epoch Rollover`

## 1. Description

Modify the client-side commit generation logic for `set` and `mset` to be epoch-aware. All new data commits must be tagged with the current epoch, and the first commit of a new epoch must link to the previous one.

## 2. Acceptance Criteria

- Before generating a commit, the client logic reads the current epoch name from the `refs/kv-epoch/<ns>` ref.
- Every new ledger commit created by `git-kv` **must** include an `Epoch: <current_epoch_name>` trailer.
- When creating the very first commit in a new epoch, the commit **must** also include a `Parent-Epoch: <previous_epoch_name>` trailer.
- The chunk writing logic must use the current epoch name to determine which chunk store ref to use (e.g., `refs/kv-chunks/<ns>@<current_epoch_name>`).

## 3. Test Plan

- **Integration Test:** After an epoch rollover, create a new key with `git kv set`. Fetch the created commit object and inspect its message to verify it contains the correct `Epoch` trailer.
- **Integration Test:** Inspect the first commit of the new epoch and verify it contains the correct `Parent-Epoch` trailer.
- **Unit Test:** Write a unit test for the commit generation function that mocks the current epoch and verifies the correct trailers are added to the commit message.
