# Task: `get` Command Reconstruction Logic

**Feature:** `US-5 — Chunked values without LFS`

## 1. Description

Modify the `git kv get` command to enable reading of chunked values. The command must be able to detect a manifest file, parse it, and reconstruct the original file by fetching and concatenating the individual chunks.

## 2. Acceptance Criteria

- The `get` command logic first fetches the object for the requested key.
- Manifest detection follows a deterministic precedence:
  1. If the key or blob path ends with `.manifest.json`, treat it as a manifest and attempt to parse and validate the JSON against the manifest schema. Failure to parse or validate must raise a manifest validation error (do not fall back to treating it as raw data).
  2. Otherwise, attempt to parse the blob content as JSON; only if it validates against the manifest schema should it be treated as a manifest. If parsing fails or the schema check fails, treat the object as a regular (non-manifest) value.
- If detected as a manifest, the logic parses JSON that must conform to this schema:
  - Top-level object with required fields:
    - `version` (string): currently only `"1"` is supported; future versions may extend the schema but unknown versions must raise an error.
    - `chunks` (array of objects, length ≥ 1).
  - Each chunk object must include:
    - `index` (integer ≥ 0). Indices must be unique and contiguous starting at 0.
    - `blob_oid` (string) matching `^[0-9a-f]{40}|[0-9a-f]{64}$`.
    - `size` (integer ≥ 0) representing the chunk byte length.
  - Optional fields:
    - `total_size` (integer ≥ 0) summarizing all chunks.
    - `checksum` (object) with optional `alg` (string) and `value` (string) for the full payload checksum.
  - Additional top-level or chunk fields are ignored for forward compatibility but must not violate the required structure.
  - Validation errors (missing fields, wrong types, non-contiguous indices, unsupported version, invalid OIDs) must surface as manifest validation errors.
  - Example manifest:
```json
{
  "version": "1",
  "total_size": 5242880,
  "chunks": [
    { "index": 0, "blob_oid": "0123456789abcdef0123456789abcdef01234567", "size": 1048576 },
    { "index": 1, "blob_oid": "89abcdef0123456789abcdef0123456789abcdef", "size": 1048576 },
    { "index": 2, "blob_oid": "fedcba9876543210fedcba9876543210fedcba98", "size": 3145728 }
  ],
  "checksum": { "alg": "sha256", "value": "aabbcc..." }
}
```
- It then iterates through the `chunks` array in ascending `index` order, verifying indices are contiguous and sizes sum to `total_size` (when provided).
- For each chunk, it fetches the corresponding Git blob using its `blob_oid`.
- The content of the blobs are written to the output stream in the correct order (by chunk index `i`).
- The entire process is transparent to the user, who simply receives the full, reconstructed file content.

## 3. Test Plan

- **Integration Test (Round Trip):** `set` a large file so it chunk-manifests, then `get` the same key and compare checksums.
- **Unit Test (Nominal Reconstruction):** Provide a mock manifest and chunk provider; assert chunks are requested sequentially by `index` and output order matches.
- **Unit Test (Missing Blob):** Manifest references a `blob_oid` absent from the store; chunk fetch should raise a descriptive error (e.g., `Missing chunk blob <oid>`).
- **Unit Test (Invalid OID):** Manifest contains an invalid `blob_oid` format; validation should fail before any fetch.
- **Unit Test (Non-sequential Indices):** Manifest indices `[0,2]` or out-of-order entries; ensure validation errors on gaps and that reordering by index occurs before streaming.
- **Unit Test (Empty Chunks Array):** Manifest with `chunks: []` must fail validation with an explicit message.
- **Integration Test (Corrupted Manifest):** Tamper with manifest JSON (missing fields) and ensure `get` surfaces validation errors.
- **Integration Test (Missing Blob):** Delete one referenced blob after manifest creation; `get` should abort and report missing chunk while leaving prior chunks untouched.
- **Integration Test (Corrupt Blob Content):** Inject corrupted content for one blob; `get` should emit an error indicating checksum/expected size mismatch if `size` is violated.
- **Integration Test (Large Manifest):** Create manifest with >10,000 chunks to exercise streaming; run `get` while monitoring RSS to confirm memory remains bounded (<256 MB) and total runtime stays within configured timeout.
- **Integration Test (Concurrent Modification):** During reconstruction, delete or replace a chunk blob mid-read to simulate race; `get` should detect the inconsistency (e.g., missing chunk on subsequent fetch) and return a retryable error.
- **Integration Test (Partial Failure Aggregation):** Combine missing and corrupt blobs; ensure the error message aggregates failing chunk indices/OIDs.
- **Performance Test:** Measure throughput/resident memory when reconstructing multi-GB payloads; ensure streaming prevents OOM and document acceptable limits (e.g., reconstruct 5GB within 60s, memory <256MB).
