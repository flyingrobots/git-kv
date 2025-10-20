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
    - Manifests themselves must not exceed 64 KB. Pushes with manifests exceeding this threshold are rejected.
  - The hook rejects pushes when either individual chunks exceed `max_value_inline` or the combined chunk sum exceeds `max_value_total`.
- **LFS Prevention:**
  - If the policy includes `forbidden: [{ lfs: true }]`, the hook inspects the pushed content.
  - It looks for Git LFS pointers (which are small text files with a specific format).
  - If an LFS pointer is detected in a path managed by `git-kv`, the push is rejected.
- Rejection messages must include:
  1. Which limit was exceeded (e.g., "max_value_inline", "max_value_total", "LFS forbidden").
  2. The policy threshold value.
  3. The actual size or constraint violated.
  4. The key path or file path involved.

  Example messages:
  - "Rejected: Key 'users/123' chunk size (2MB) exceeds max_value_inline (1MB)."
  - "Rejected: Key 'data/x' total chunked value size (50MB) exceeds max_value_total (20MB)."
  - "Rejected: LFS pointer detected at 'assets/video.mp4' but LFS is forbidden by policy."

## 3. Test Plan

- **Unit Test (Size Validation Logic):**
  - Test chunked value detection all three methods (manifest suffix, ledger entry, refs).
  - Test size comparison for chunks at boundary conditions (== limit, > limit, < limit).
  - Test manifest size validation (exactly at threshold, over threshold).

- **Integration Test (Size Limits):**
  - Set `max_value_inline: 1MB, max_value_total: 100MB` in policy.
  - Scenario 1: Push a single chunk of 1.5 MB → Assert rejection with message including "max_value_inline", "1MB", and key path.
  - Scenario 2: Push 50 chunks of 2.5 MB each (250 MB total) → Assert rejection with message including "max_value_total", "100MB", and total size.
  - Scenario 3: Push 40 chunks of 2 MB each (80 MB total) → Assert acceptance.
  - Scenario 4: Push manifest >64 KB → Assert rejection.

- **Integration Test (LFS Prevention):**
  - Enable `forbidden: [{ lfs: true }]` in policy.
  - Scenario 1: Commit LFS pointer at `managed/namespace/file.bin` → Assert rejection with "LFS forbidden" message.
  - Scenario 2: Commit LFS pointer at `external/file.bin` (outside managed paths) → Assert acceptance.
  - Scenario 3: Disable LFS prevention; commit LFS pointer at managed path → Assert acceptance.

- **Negative Test (Invalid Manifest):**
  - Push a manifest with missing or invalid chunk refs → Assert graceful failure with diagnostic message.
