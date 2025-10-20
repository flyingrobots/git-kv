# Task: Epoch-Aware Commit Generation

**Feature:** `US-6 — Epoch Rollover`

## 1. Description

Modify the client-side commit generation logic for `set` and `mset` to be epoch-aware. All new data commits must be tagged with the current epoch, and the first commit of a new epoch must link to the previous one.

## 2. Acceptance Criteria

- Before generating a commit, the client logic reads the current epoch name from the `refs/kv-epoch/<ns>` ref. This read **must** be performed immediately before the push operation to minimize staleness.
- Every new ledger commit created by `git-kv` **must** include an `Epoch: <current_epoch_name>` trailer.
- When creating the very first commit in a new epoch, the commit **must** also include a `Parent-Epoch: <previous_epoch_name>` trailer.
- The chunk writing logic must use the current epoch name to determine which chunk store ref to use (e.g., `refs/kv-chunks/<ns>@<current_epoch_name>`).
- **Concurrency Handling:** If the epoch changes between the client reading the epoch and pushing the commit, the Stargate `pre-receive` hook **must** reject the commit if the `Epoch:` trailer does not match the current Stargate epoch. Clients are expected to re-read the epoch and retry the operation.

## 3. Test Plan

- **Integration Test:** After an epoch rollover, create a new key with `git kv set`. Fetch the created commit object and inspect its message to verify it contains the correct `Epoch` trailer.
- **Integration Test:** Inspect the first commit of the new epoch and verify it contains the correct `Parent-Epoch` trailer.
- **Unit Test:** Write a unit test for the commit generation function that mocks the current epoch and verifies the correct trailers are added to the commit message.
- **Integration Test (Epoch Rollover Mid-Operation):**
  1. Start a `git kv mset` operation on the client.
  2. While the client is processing (e.g., after tree construction but before push), trigger an `git kv epoch new` on the Stargate.
  3. Assert that the client's push is rejected by the Stargate due to an epoch mismatch.
  4. Assert that the client handles the rejection gracefully (e.g., by retrying the operation after re-reading the current epoch).
