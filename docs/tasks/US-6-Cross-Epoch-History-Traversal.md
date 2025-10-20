# Task: Cross-Epoch History Traversal

**Feature:** `US-6 — Epoch Rollover` (and `US-9`)

## 1. Description

Update the `git kv history <key>` command to support traversing a key's history across multiple epochs. This is essential for providing a complete audit trail.

## 2. Acceptance Criteria

- The `history` command logic fetches the commit log for a key within the current epoch.
- The final output presented to the user is a single, continuous history log, ordered reverse-chronologically (newest-to-oldest) by commit timestamp.
- When it reaches the first commit of an epoch, it inspects the commit message for a `Parent-Epoch` trailer.
- If the `Parent-Epoch` trailer exists and points to a valid, accessible parent epoch ref, the logic recursively continues fetching the key's history from the parent epoch's ledger ref.
- If the `Parent-Epoch` trailer points to a deleted/non-existent ref, the command **must** surface a deterministic error message (e.g., "Error: Parent epoch ref <ref> not found") and stop traversal.
- If a `Parent-Epoch` trailer is malformed, the command **must** report a parse error and abort.
- If a cycle in `Parent-Epoch` links is detected, the command **must** abort with a clear cycle-detection error or enforce a maximum traversal depth (e.g., 100 epochs) to prevent infinite loops.

## 3. Test Plan

- **Integration Test:** Create a key in epoch A. Roll over to epoch B and update the key. Roll over to epoch C and update the key again. Run `git kv history` on the key and verify the output contains the commits from all three epochs in the correct order.
- **Unit Test:** Write a unit test for the traversal logic that uses mock commit objects with and without `Parent-Epoch` trailers.
- **Edge Case (Non-existent Parent Ref):** Simulate a `Parent-Epoch` trailer pointing to a non-existent ref. Assert the command reports a deterministic error and stops traversal.
- **Edge Case (Malformed Parent Trailer):** Simulate a malformed `Parent-Epoch` trailer. Assert the command reports a parse error and aborts.
- **Edge Case (Epoch Cycle):** Simulate a circular `Parent-Epoch` link. Assert the command detects the cycle and aborts with an error.
- **Normal Cases:** Test with and without `Parent-Epoch` trailers to ensure correct behavior.
