# Task: Stargate Attestation Validation Logic

**Feature:** `US-G2 — Atomic transaction via Stargate`

## 1. Description

This is the core of the Stargate's security model. Implement the O(1) validation logic that runs inside the `pre-receive` hook. This logic inspects the attestation trailers of the pushed commits and verifies their integrity without needing to inspect the full content of the trees.

## 2. Acceptance Criteria

- The validation logic receives the details of the refs being pushed.
- It finds the pushed ledger and index commits that share the same `KV-Txn` trailer.
- It reads the `KV-Index-Tree` trailer from the ledger commit's message.
- It compares this value to the actual tree OID of the pushed index commit.
- It reads the `KV-Ledger-Tree` trailer from the index commit's message.
- It compares this value to the actual tree OID of the pushed ledger commit.
- The push is rejected if any of these cross-references do not match.
- The logic must also perform a fast-forward check to ensure linear history.

## 3. Test Plan

- **Unit Test (Success):** Provide a pair of mock ledger and index commits with correct, matching trailers. Verify the validation passes.
- **Unit Test (Failure):** Provide a pair of mock commits where the `KV-Index-Tree` trailer in the ledger commit does not match the actual index tree OID. Verify the validation fails.
- **Unit Test (Non-FF):** Simulate a non-fast-forward push and verify it is rejected.
- **Unit Test (Missing Trailers):** Test with commits that are missing the required trailers and verify validation fails.
