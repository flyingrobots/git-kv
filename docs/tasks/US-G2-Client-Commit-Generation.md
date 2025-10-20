# Task: Client-Side Commit Generation for `mset`

**Feature:** `US-G2 — Atomic transaction via Stargate`

## 1. Description

Implement the logic to create the new ledger and index commits after the trees have been constructed. This is a critical step that embeds the attestation trailers into the commit messages, which the Stargate uses for O(1) validation.

## 2. Acceptance Criteria

- A function takes the new ledger tree OID and index tree OID as input.
- It generates a new ledger commit pointing to the new ledger tree.
- The ledger commit message **must** contain the required attestation trailers, including:
  - `KV-NS`
  - `KV-Txn` (a new unique transaction ID)
  - `Base-Tree`
  - `KV-Index-Tree` (must match the OID of the new index tree)
  - `Epoch`
- It generates a new index commit pointing to the new index tree.
- The index commit message **must** contain `KV-Ledger-Tree` and `KV-Txn` trailers.

## 3. Test Plan

- **Unit Test:** Call the function with mock tree OIDs. Verify that the returned commit objects have the correct parent, tree, and that the commit messages contain the correctly formatted attestation trailers.
- **Unit Test:** Ensure a new, unique `KV-Txn` ID is generated for each call.
- **Edge Case:** Test with placeholder or empty OIDs to ensure robust error handling.
