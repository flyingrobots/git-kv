# Task: Cross-Epoch History Traversal

**Feature:** `US-6 — Epoch Rollover` (and `US-9`)

## 1. Description

Update the `git kv history <key>` command to support traversing a key's history across multiple epochs. This is essential for providing a complete audit trail.

## 2. Acceptance Criteria

- The `history` command logic fetches the commit log for a key within the current epoch.
- When it reaches the first commit of the epoch, it inspects the commit message for a `Parent-Epoch` trailer.
- If the trailer exists, the logic identifies the parent epoch's ledger ref.
- It then recursively continues fetching the key's history from the parent epoch's ref.
- The final output presented to the user is a single, continuous history log, correctly ordered.

## 3. Test Plan

- **Integration Test:** Create a key in epoch A. Roll over to epoch B and update the key. Roll over to epoch C and update the key again. Run `git kv history` on the key and verify the output contains the commits from all three epochs in the correct order.
- **Unit Test:** Write a unit test for the traversal logic that uses mock commit objects with and without `Parent-Epoch` trailers.
