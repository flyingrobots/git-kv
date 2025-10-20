# Task: `set` Command Large Value Logic

**Feature:** `US-5 — Chunked values without LFS`

## 1. Description

Modify the `git kv set` command to automatically handle large values. When a value is provided that exceeds the `max_value_inline` threshold, the command should trigger the chunking workflow instead of storing the value directly.

## 2. Acceptance Criteria

- The `set` command logic checks the size of the input value (from a file or stdin).
- If the size is greater than `max_value_inline` from the policy file, the chunking process is initiated.
- The chunks are created as Git blobs and stored in a new `refs/kv-chunks/<ns>@<epoch>` tree.
- A manifest file is created in the main ledger tree instead of the raw value.
- The process is transparent to the user unless an error occurs.

## 3. Test Plan

- **Integration Test:** Run `git kv set` with a file larger than the inline threshold. Verify that a `.manifest.json` file is created in the ledger, not the raw file content.
- **Integration Test:** Inspect the `refs/kv-chunks/...` ref and verify that it contains new Git blobs corresponding to the chunks.
- **Failure Case:** Test with a file that exceeds the `max_value_total` and verify the command fails with an appropriate error.
