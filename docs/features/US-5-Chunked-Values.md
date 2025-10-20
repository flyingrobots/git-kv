# Feature Spec: US-5 — Chunked values without LFS

**User Story:** As a developer, I can work with large values (>1MB) without needing to use Git LFS.

**Depends On:** `S` (Chunks) writer/reader; manifest format.

## 1. Acceptance Criteria

- The `git kv set` command can accept values larger than the `max_value_inline` threshold (e.g., 1MB).
- When a large value is provided, the client automatically chunks the file using FastCDC.
- The individual chunks are stored as standard Git blobs in a dedicated `refs/kv-chunks/<ns>@<epoch>` ref.
- Instead of the value, a manifest file (`.manifest.json`) is written to the `refs/kv/<ns>` (Ledger) branch.
- This manifest contains the list of chunk digests and their corresponding Git blob OIDs.
- The `git kv get <key>` command can read chunked values.
- When `get` is called on a key with a manifest, the client reads the manifest, fetches all the chunk blobs from Git, and reconstructs the original file.
- The process is transparent to the user.

## 2. Test Plan

- **Round Trip:** `set` a file larger than 1MB. `get` the file back and verify its content is identical to the original (e.g., by comparing checksums).
- **Deduplication:** `set` a large file. Then, `set` a slightly modified version of the same file. Verify that only the new/changed chunks are added to the chunk store, demonstrating deduplication.
- **Manifest Correctness:** `set` a large file and then manually inspect the created manifest file in the ledger branch. Verify its format and content are correct.
- **`get` Performance:** Measure the performance of `get` for a large, multi-chunk file.

## 3. Tasks

- [ ] **Chunking Library:** Choose and integrate a Go library for FastCDC.
- [ ] **Client: `set` Logic:** Modify the `set` command to check the input value size against `max_value_inline`.
- [ ] **Client: Chunker:** If the value is large, implement the logic to chunk the file, create Git blobs for each chunk, and store them in the `refs/kv-chunks/...` ref.
- [ ] **Client: Manifest Writer:** Implement logic to generate and write the `.manifest.json` file to the ledger.
- [ ] **Client: `get` Logic:** Modify the `get` command to detect if a key represents a manifest file.
- [ ] **Client: Reconstructor:** If it's a manifest, implement logic to read the manifest, fetch all chunk blobs, and stream them back to the user in the correct order.
- [ ] **Testing:** Add unit tests for the chunking and reconstruction logic.
- [ ] **Testing:** Add an integration test for the large file round-trip scenario.
- [ ] **Documentation:** Document the automatic large value handling feature.
