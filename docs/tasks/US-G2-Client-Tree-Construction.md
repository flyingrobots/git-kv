# Task: Client-Side Tree Construction for `mset`

**Feature:** `US-G2 — Atomic transaction via Stargate`

## 1. Description

Implement the client-side logic to process a batch file for `git kv mset`. This involves reading a JSONL file where each line represents a `set` or `del` operation, and using this information to construct new Git trees for both the ledger (`refs/kv/<ns>`) and the index (`refs/kv-index/<ns>`) in memory.

## 2. Acceptance Criteria

- The logic correctly parses a valid JSONL file.
- For each `set` operation, a new blob is created for the value, and the corresponding path in the ledger tree is updated to point to it. The index tree is also updated with a new `.ref` pointer.
- For each `del` operation, the corresponding path is removed from the ledger tree, and the index tree is updated with a tombstone `.ref` pointer.
- The function returns the OIDs of the newly constructed ledger tree and index tree.
- The logic handles file paths and key hashing (`<hh>/<hh>/<urlsafe-key>`) correctly.

## 3. Test Plan

- **Unit Test (Set):** Provide a sample batch file with only `set` operations. Verify that the resulting tree OIDs are correct and that the in-memory tree structure matches the expected layout.
- **Unit Test (Del):** Provide a sample batch file with `del` operations. Verify the correct paths are removed and tombstones are created in the index.
- **Unit Test (Mixed):** Test with a mix of `set` and `del` operations.
- **Edge Case:** Test with a malformed JSONL file and ensure the function returns a descriptive error.
- **Edge Case:** Test with an empty batch file; the function should return the original, unchanged tree OIDs.
