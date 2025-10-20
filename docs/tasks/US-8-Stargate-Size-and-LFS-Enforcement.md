# Task: Stargate Size and LFS Enforcement

**Feature:** `US-8 — Policy Enforcement`

## 1. Description

Implement two additional policy enforcement checks within the Stargate `pre-receive` hook: value size limits and Git LFS prevention.

## 2. Acceptance Criteria

- **Dependency:** This task depends on `US-5-Integrate-Chunking-Library.md` and `US-5-Set-Command-Large-Value-Logic.md` for chunking mechanism details.
- **Size Limits:**
  - The hook inspects the size of blobs for all `set` operations in a transaction.
  - **Chunked Value Detection:** The hook must detect chunked values by either recognizing a `.manifest.json` suffix on the key path, inspecting the ledger tree object for a manifest entry, or verifying the existence of corresponding refs under `refs/kv-chunks/<ns>@<epoch>`.
  - **Size Validation Semantics:**
    - `max_value_inline` applies to any single inline blob or single chunk.
    - `max_value_total` applies to the sum of all chunk sizes referenced by a manifest.
    - Manifests themselves must be below a small metadata-size threshold (e.g., 1MB).
  - The hook rejects pushes when either individual chunks exceed `max_value_inline` or the combined chunk sum exceeds `max_value_total`.
- **LFS Prevention:**
  - If the policy includes `forbidden: [{ lfs: true }]`, the hook inspects the pushed content.
  - It looks for Git LFS pointers (which are small text files with a specific format).
  - If an LFS pointer is detected in a path managed by `git-kv`, the push is rejected.
- Rejection messages must be clear about which limit was exceeded.

## 3. Test Plan

- **Integration Test (Size):** Set a low `max_value_total` in the policy. Attempt to push a key with a value larger than this limit and verify it is rejected.
- **Integration Test (LFS):** Enable LFS prevention. Commit a file using Git LFS under a `git-kv` managed path. Attempt to push this commit and verify it is rejected by the Stargate hook.
