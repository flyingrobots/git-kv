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
  - The hook inspects three categories of keys:
    1. Commit signer keys extracted from author/committer signatures.
    2. Public keys embedded in configuration/artifact files touched by the commit (files matching policy globs such as `config/keys/*.pub`).
    3. Public keys referenced in commit metadata (trailers or fields explicitly named `Key-Ref`, `Parent-Key`, etc.).
  - Each key discovered in these categories must start with one of the configured `allowed_prefixes`; otherwise the transaction is rejected.
  - Example: a commit touching `config/keys/app1.pub` and referencing `Key-Ref: ssh-ed25519 AAA...` must validate both the signer key and the referenced key against `allowed_prefixes`.
- Rejection messages **must** be a single-line string in the exact format: `POLICY_VIOLATION:<ERROR_CODE>:<POLICY_NAME>:<DETAIL>`.
  - `ERROR_CODE`: A short uppercase token (e.g., `UNSIGNED_COMMIT`, `FORBIDDEN_PREFIX`).
  - `POLICY_NAME`: The identifier of the violated policy (e.g., `require_signed_commits`, `allowed_prefixes`).
  - `DETAIL`: A human-readable brief explanation.
  - **Example:** `POLICY_VIOLATION:UNSIGNED_COMMIT:require_signed_commits:push signed by unauthorized key`
- Implementations **must** emit messages exactly in this pattern so clients can reliably parse by splitting on `:` and validating the first token equals `POLICY_VIOLATION`.

## 3. Test Plan

- **Integration Test (Signer Policy):**
  - **Case 1 (Unsigned Commit):** Enable `require_signed_commits`. Push an unsigned commit. Assert rejection with `POLICY_VIOLATION:UNSIGNED_COMMIT:require_signed_commits:...`.
  - **Case 2 (Unauthorized Signer):** Push a commit signed by an unauthorized key. Assert rejection with `POLICY_VIOLATION:UNAUTHORIZED_SIGNER:require_signed_commits:...`.
  - **Case 3 (Authorized Signer):** Push a commit signed by an authorized key. Assert success.
- **Integration Test (Prefix Policy):**
  - Define a strict `allowed_prefixes` list. Push a transaction containing a key with a forbidden prefix. Assert rejection with `POLICY_VIOLATION:FORBIDDEN_PREFIX:allowed_prefixes:...`.
