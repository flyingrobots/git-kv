# Task: `set` Command Large Value Logic

**Feature:** `US-5 — Chunked values without LFS`

## 1. Description

Modify the `git kv set` command to automatically handle large values. When a value is provided that exceeds the `max_value_inline` threshold, the command should trigger the chunking workflow instead of storing the value directly.

## 2. Acceptance Criteria

- The `set` command logic checks the size of the input value (from a file or stdin).
- If the size is greater than `max_value_inline` (defaulting to 1MB if absent in policy, with a warning logged), the chunking process is initiated.
- The chunks are created as Git blobs and stored in a tree at `refs/kv-chunks/<ns>@<epoch>`. Each chunk is a blob child named by a zero-padded sequence number (e.g., `000001`, `000002`, ...) and referenced in a tree listing, not nested in subdirectories.
- A manifest file is created in the main ledger tree as a sibling object at `<key-path>/.manifests/<key>-<epoch>.json` containing ordered chunk sequence and blob hashes.
- Cross-reference `US-5-Integrate-Chunking-Library.md` for min/avg/max chunk sizes.
- The process is transparent to the user unless an error occurs.

## 3. Test Plan

- **Integration Test (Chunking Success):**
  - **Fixture:** `docs/tasks/testdata/chunker_fixtures/large-file-1mb.bin` (1,048,576 bytes).
  - **Chunking Policy:** Use `minSize=64KB`, `avgSize=256KB`, `maxSize=1MB` (as defined in `US-5-Integrate-Chunking-Library.md`).
  - **Expected Chunks:** Compute the expected number of chunks based on the fixture size and chunking policy.
  - Run `git kv set` with the fixture file. Verify that a manifest file is created in the ledger, not the raw file content.
  - Inspect the `refs/kv-chunks/...` ref and verify it contains the **exact expected number** of new Git blobs corresponding to the chunks.
- **Failure Case (Exceed `max_value_total`):**
  - **Policy:** Set `max_value_total` to 1.5MB in the test policy (originating from `.kv/policy.yaml`).
  - **Fixture:** Use a file larger than 1.5MB.
  - Run `git kv set` with this file. Verify the command fails with an error indicating the `max_value_total` limit was exceeded.
