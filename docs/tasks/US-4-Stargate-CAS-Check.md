# Task: Stargate CAS Check in `pre-receive`

**Feature:** `US-4 — CAS on branch`

## 1. Description

Update the Stargate's `pre-receive` hook to perform the Compare-And-Swap (CAS) check. When a client provides the `--expect-tree` value, the hook must verify that the current state of the branch matches the expected state before allowing the update.

## 2. Acceptance Criteria

- The `pre-receive` hook needs a mechanism to receive the `--expect-tree` OID from the client. (This could be a commit trailer or a push option).
- Before other validation, the hook checks if a CAS operation is requested.
- If so, it reads the current OID of the `refs/kv/<ns>` branch tip.
- It compares this current OID with the expected OID from the client.
- If they do not match, the hook rejects the push with a specific, machine-readable error message: "ERROR: CAS_CONFLICT: Expected tree OID mismatch." This rejection must also use a specific exit code (e.g., 2).
- If they do match, validation continues as normal.

## 3. Test Plan

- **Integration Test (Success):** Push a commit with a correct `--expect-tree` OID. Verify the push is accepted.
- **Integration Test (Failure):** Simulate a race condition: get the current OID, push a separate commit to update the branch, then try to push a new commit using the original OID with `--expect-tree`. Verify this second push is rejected with the specific CAS error.
