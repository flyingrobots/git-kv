# Task: Stargate Size and LFS Enforcement

**Feature:** `US-8 — Policy Enforcement`

## 1. Description

Implement two additional policy enforcement checks within the Stargate `pre-receive` hook: value size limits and Git LFS prevention.

## 2. Acceptance Criteria

- **Dependency:** This task depends on the US-5 artifacts: `US-5-Integrate-Chunking-Library.md` (FastCDC API, chunk size defaults), `US-5-Set-Command-Large-Value-Logic.md` (chunk upload endpoint contract), and `US-5-Manifest-File-Generation.md` (manifest schema). Enforcement must validate pushes against these specific contracts.
- **Size Limits:**
  - The hook inspects the size of blobs for all `set` operations in a transaction.
  - **Chunked Value Detection:** The hook must detect chunked values by either recognizing a `.manifest.json` suffix on the key path, inspecting the ledger tree object for a manifest entry, or verifying the existence of corresponding refs under `refs/kv-chunks/<ns>@<epoch>`.
  - **Size Validation Semantics:**
    - `max_value_inline` applies to any single inline blob or single chunk.
    - `max_value_total` applies to the sum of all chunk sizes referenced by a manifest.
    - Manifests themselves must not exceed 64 KB measured as the uncompressed UTF-8 JSON byte length stored in the object (excluding manifest key path/metadata). Pushes with manifests exceeding this threshold are rejected.
- The hook rejects pushes when either individual chunks exceed `max_value_inline` or the combined chunk sum exceeds `max_value_total`.
- Validation follows optimistic concurrency: each push is evaluated atomically against the current manifest/chunk set; concurrent pushes to the same key are serialized via Stargate's pre-receive queue (successful pushes advance manifests, conflicting pushes receive deterministic rejection). Garbage-collection processes must coordinate via version checks or locks so active pushes encountering GC races receive a retriable `RETRY_LATER` error; clients should retry with exponential backoff.
- The hook must confirm chunk uploads adhere to the US-5 contract: manifests reference chunk IDs matching `/chunks` API responses, each chunk size equals the FastCDC defaults (`min=64KB`, `avg=256KB`, `max=1MB` unless overridden), and uploads exceeding `max_value_inline` are chunked rather than inline.
- Documented failure modes include the canonical error codes defined in the **Error Codes & Semantics** section below (`SIZE_LIMIT_EXCEEDED`, `MANIFEST_TOO_LARGE`, `LFS_FORBIDDEN`, `RETRY_LATER`).
- **LFS Prevention:**
  - If the policy includes `forbidden: [{ lfs: true }]`, the hook inspects the pushed content.
  - It looks for Git LFS pointers (which are small text files with a specific format).
  - If an LFS pointer is detected in a path managed by `git-kv`, the push is rejected.
- Rejection messages must follow the canonical format `Rejected: <constraint>: <actual_value> vs <limit> (<human_readable_bytes>)` with uppercase units (B, KB, MB, GB).
  - Example (inline limit): `Rejected: max_value_inline: 2097152 B vs 1048576 B (2.0 MB actual, 1.0 MB limit) KEY=users/123`.
  - Example (total limit): `Rejected: max_value_total: 52428800 B vs 20971520 B (50.0 MB actual, 20.0 MB limit) KEY=data/x`.
  - Example (LFS): `Rejected: LFS: detected at 'assets/video.mp4' — forbidden by policy`.

### Error Codes & Semantics

All hook failures must emit a single-line JSON object on stderr using the schema `{ "code": string, "message": string, "details": object }` in addition to any human-readable summary. The table below defines the required contract for each code:

| Code | HTTP Status | Git Hook Exit Code | Cause | Client Action |
| --- | --- | --- | --- | --- |
| `SIZE_LIMIT_EXCEEDED` | 400 | 1 | Any inline blob or chunk exceeds `max_value_inline` or the summed chunks exceed `max_value_total`. `details` MUST include `key`, `limit_bytes`, and `actual_bytes`. | Reduce payload size or chunk appropriately before retrying. |
| `MANIFEST_TOO_LARGE` | 400 | 1 | Manifest JSON exceeds 64 KiB uncompressed. `details` MUST include `key`, `manifest_bytes`, `limit_bytes`. | Regenerate manifest within limits and retry. |
| `LFS_FORBIDDEN` | 403 | 1 | Detected Git LFS pointer in a path governed by policy. `details` MUST include `path` and `pointer_oid`. | Remove LFS pointer or disable policy before retrying (terminal). |
| `RETRY_LATER` | 503 | 75 (EX_TEMPFAIL) | Transient contention (e.g., GC races, CAS conflicts while validating chunks/manifests). `details` MUST include `reason` (e.g., `GC_RACE`, `CAS_CONFLICT`) and `retry_after_ms`. | Retry with exponential backoff (base 100ms, multiplier 2, cap 5s) up to 5 attempts. |

Operational requirements for `RETRY_LATER`:

- The hook MUST log the transient reason with structured metadata so operators can distinguish temporary vs terminal failures.
- Clients encountering `RETRY_LATER` MUST retry the push using the documented backoff profile (initial delay 100ms, double each attempt, capped at 5s, max 5 attempts). Exceeding the attempt budget escalates to operators.

**Examples:**

```json
{"code":"SIZE_LIMIT_EXCEEDED","message":"inline blob exceeds max_value_inline","details":{"key":"users/123","limit_bytes":1048576,"actual_bytes":2097152}}
```

```json
{"code":"RETRY_LATER","message":"temporary manifest validation race","details":{"reason":"GC_RACE","retry_after_ms":500}}
```

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
  - Scenario 5: Upload chunked value via US-5 `/chunks` endpoint ensuring chunk sizes respect FastCDC defaults; verify Stargate accepts and enforces totals.
  - Scenario 6: Simulate US-5 API schema change (unexpected chunk field); assert Stargate rejects with descriptive error rather than misprocessing data.

- **Integration Test (LFS Prevention):**
  - Enable `forbidden: [{ lfs: true }]` in policy.
  - Scenario 1: Commit LFS pointer at `managed/namespace/file.bin` → Assert rejection with "LFS forbidden" message.
  - Scenario 2: Commit LFS pointer at `external/file.bin` (outside managed paths) → Assert acceptance.
  - Scenario 3: Disable LFS prevention; commit LFS pointer at managed path → Assert acceptance.

- **Negative Test (Invalid Manifest):**
  - Push a manifest with missing or invalid chunk refs → Assert graceful failure with diagnostic message.
- **Integration Test (Concurrency):** Simultaneous pushes to same key (one inline, one chunked) should result in deterministic acceptance/rejection consistent with last-writer-wins or `RETRY_LATER` error when CAS fails.
- **Integration Test (GC Interaction):** Run garbage-collection concurrently with large push; ensure push either succeeds or receives documented `RETRY_LATER` error and no partial state remains.
