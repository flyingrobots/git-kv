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

- **Integration Test:** Run `git kv set` with a file larger than the inline threshold. Verify that a `.manifest.json` file is created in the ledger, not the raw file content.
- **Integration Test:** Inspect the `refs/kv-chunks/...` ref and verify that it contains new Git blobs corresponding to the chunks.
- **Failure Case:** Test with a file that exceeds the `max_value_total` and verify the command fails with an appropriate error.
