# Task: Cross-Epoch History Traversal

**Feature:** `US-6 — Epoch Rollover` (and `US-9`)

## 1. Description

Update the `git kv history <key>` command to support traversing a key's history across multiple epochs. This is essential for providing a complete audit trail.

## 2. Acceptance Criteria

- The `history` command logic fetches the commit log for a key within the current epoch.
- The command collects candidate commits for the key from every reachable epoch, materializes them into a single list, normalizes commit timestamps to UTC, and performs a global sort descending by timestamp. If two commits share the same timestamp, break ties deterministically using (1) an epoch sequence number (newer epoch first) and (2) commit hash lexicographical order. This ensures a stable ordering even under clock skew; implementers may log a warning when skew exceeding a configured threshold is detected. Note that this requires O(n log n) sorting over all candidate commits—concatenating per-epoch lists without sorting is not permitted. The resulting order is returned as the continuous history log.
- The traversal treats an epoch boundary as the earliest commit for the key whose parent commit lives in a different `refs/kv/<epoch>` branch (or is missing). Detection uses commit ancestry: walk parents within the current epoch branch; when the next parent is absent or references another epoch, mark the current commit as the "first commit" of this epoch. If timestamps are equal, rely on the epoch sequence number and commit hash tie-breaker to choose the boundary. Once at this boundary, inspect the commit message for a `Parent-Epoch` trailer.
- If the `Parent-Epoch` trailer exists and points to a valid, accessible parent epoch ref, the logic recursively continues fetching the key's history from the parent epoch's ledger ref.
- If the `Parent-Epoch` trailer points to a deleted/non-existent ref, the command **must** surface a deterministic error message (e.g., "Error: Parent epoch ref <ref> not found") and stop traversal.
- `Parent-Epoch` trailers must conform to Git trailer syntax (`Key: Value` per https://git-scm.com/docs/git-interpret-trailers). Valid examples: `Parent-Epoch: 42`, `Parent-Epoch: 0007`. Invalid examples include missing colon (`Parent-Epoch 42`), misspelled key (`ParentEpoch: 42`), double colon (`Parent-Epoch:: 42`), empty value (`Parent-Epoch:`), non-numeric or negative values (`Parent-Epoch: abc`, `Parent-Epoch: -1`), or values containing leading/trailing spaces unless they are percent-encoded. Duplicate `Parent-Epoch` trailers are an error. Values must be ASCII digits representing an integer ≥ 0.
- If a `Parent-Epoch` trailer violates these rules, the command **must** report a parse error and abort.
- If a cycle in `Parent-Epoch` links is detected, the command **must** abort with a clear cycle-detection error or enforce a maximum traversal depth (e.g., 100 epochs) to prevent infinite loops.

## 3. Test Plan

- **Integration Test:** Create a key in epoch A. Roll over to epoch B and update the key. Roll over to epoch C and update the key again. Run `git kv history` on the key and verify the output contains the commits from all three epochs in the correct order.
- **Unit Test:** Write a unit test for the traversal logic that uses mock commit objects with and without `Parent-Epoch` trailers.
- **Edge Case (Non-existent Parent Ref):** Simulate a `Parent-Epoch` trailer pointing to a non-existent ref. Assert the command reports a deterministic error and stops traversal.
- **Edge Case (Malformed Parent Trailer):** Simulate a malformed `Parent-Epoch` trailer. Assert the command reports a parse error and aborts.
- **Edge Case (Epoch Cycle):** Simulate a circular `Parent-Epoch` link. Assert the command detects the cycle and aborts with an error.
- **Edge Case (Key Missing in Parent):** Create a parent epoch ref without the key while the child epoch references it; assert traversal stops with a deterministic error (e.g., "Error: Key <k> not found in parent epoch <ref>") or other documented behavior.
- **Edge Case (Permission Denied):** Mock ref access to throw a permission error; assert the command surfaces "Error: Unable to read parent epoch <ref> (permission denied)" and halts.
- **Edge Case (Empty Intermediate Epochs):** Construct epochs where the key has no commits. Assert traversal either skips them silently or emits the documented warning (`Warning: No commits for key <k> in epoch <ref>`), matching the chosen implementation contract.
- **Edge Case (Timestamp Collision):** Create commits across epochs with identical timestamps and verify output ordering respects the timestamp sort with epoch/commit tie-breaker.
- **Performance Test (Large History):** Generate thousands of commits across >50 epochs and assert traversal completes within the performance SLA (e.g., <5s, memory <256MB).
- **Concurrency Test:** Simulate concurrent traversal where parent epochs change mid-read; ensure traversal either re-evaluates or reports a stable view error.
- **Normal Cases:** Test with and without `Parent-Epoch` trailers to ensure correct behavior.
