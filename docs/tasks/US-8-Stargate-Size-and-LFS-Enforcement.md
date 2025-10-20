# Task: Stargate Size and LFS Enforcement

**Feature:** `US-8 — Policy Enforcement`

## 1. Description

Implement two additional policy enforcement checks within the Stargate `pre-receive` hook: value size limits and Git LFS prevention.

## 2. Acceptance Criteria

- **Size Limits:**
  - The hook inspects the size of blobs for all `set` operations in a transaction.
  - It rejects the push if a blob's size exceeds `max_value_inline` (and it's not being chunked) or if it exceeds `max_value_total`.
- **LFS Prevention:**
  - If the policy includes `forbidden: [{ lfs: true }]`, the hook inspects the pushed content.
  - It looks for Git LFS pointers (which are small text files with a specific format).
  - If an LFS pointer is detected in a path managed by `git-kv`, the push is rejected.
- Rejection messages must be clear about which limit was exceeded.

## 3. Test Plan

- **Integration Test (Size):** Set a low `max_value_total` in the policy. Attempt to push a key with a value larger than this limit and verify it is rejected.
- **Integration Test (LFS):** Enable LFS prevention. Commit a file using Git LFS under a `git-kv` managed path. Attempt to push this commit and verify it is rejected by the Stargate hook.
