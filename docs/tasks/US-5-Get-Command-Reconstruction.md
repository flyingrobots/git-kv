# Task: `get` Command Reconstruction Logic

**Feature:** `US-5 — Chunked values without LFS`

## 1. Description

Modify the `git kv get` command to enable reading of chunked values. The command must be able to detect a manifest file, parse it, and reconstruct the original file by fetching and concatenating the individual chunks.

## 2. Acceptance Criteria

- The `get` command logic first fetches the object for the requested key.
- It inspects the object to see if it is a manifest file (e.g., by checking for a `.manifest.json` suffix or by parsing the content).
- If it is a manifest, the logic parses the JSON.
- It then iterates through the `chunks` array in the manifest.
- For each chunk, it fetches the corresponding Git blob using its `blob_oid`.
- The content of the blobs are written to the output stream in the correct order (by chunk index `i`).
- The entire process is transparent to the user, who simply receives the full, reconstructed file content.

## 3. Test Plan

- **Integration Test (Round Trip):** This is the most critical test. `set` a large file, which will be chunked. Then `get` the same key and compare the checksum of the output with the original file. They must match.
- **Unit Test:** Write a unit test for the reconstruction logic. Give it a mock manifest and a function to provide mock chunk data. Verify that it requests the correct chunks in the correct order.
- **Failure Case:** Test `get` on a key with a corrupted or incomplete manifest. The command should fail with a clear error.
