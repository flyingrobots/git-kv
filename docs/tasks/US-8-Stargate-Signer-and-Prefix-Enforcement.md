# Task: Stargate Signer and Prefix Enforcement

**Feature:** `US-8 — Policy Enforcement`

## 1. Description

Implement two key policy enforcement checks within the Stargate `pre-receive` hook: signer verification and allowed key prefixes.

## 2. Acceptance Criteria

- **Signer Verification:**
  - If `require_signed_commits` is true in the policy, the hook uses `git verify-commit` on the pushed commit.
  - It extracts the signer's key (GPG or SSH).
  - The push is rejected if the signature is invalid or if the signer's key is not in the policy's `writers` list.
- **Prefix Enforcement:**
  - The hook inspects all keys involved in the transaction.
  - For each key, it checks if the key starts with one of the prefixes in the `allowed_prefixes` list.
  - If any key does not match, the entire transaction is rejected.
- Rejection messages must be clear and indicate which policy was violated.

## 3. Test Plan

- **Integration Test (Signer):** Enable `require_signed_commits`. Push an unsigned commit and verify rejection. Push a commit signed by an unauthorized key and verify rejection. Push a commit signed by an authorized key and verify success.
- **Integration Test (Prefix):** Define a strict `allowed_prefixes` list. Push a transaction containing a key with a forbidden prefix and verify rejection.
