# Task: CLI `--expect-tree` Flag

**Feature:** `US-4 — CAS on branch`

## 1. Description

Add the `--expect-tree <OID>` flag to the `git kv set` and `git kv mset` commands. This flag is the entry point for the user to initiate a Compare-And-Swap operation.

## 2. Acceptance Criteria

- The `set` command in the CLI is updated to accept a new string flag, `--expect-tree`.
- The `mset` command in the CLI is updated to accept the same `--expect-tree` flag.
- The value of the flag (the OID) is captured and passed to the underlying client logic.
- If the provided OID is not a valid Git OID format, the CLI should exit with a validation error before attempting any Git operations.

## 3. Test Plan

- **Unit Test:** Test the CLI parsing to ensure the flag is correctly registered and its value is passed to the application logic.
- **Integration Test:** Run `git kv set --expect-tree 12345` and verify the CLI exits with an error because the OID is malformed.
- **Integration Test:** Run `git kv set --help` and verify the new flag is present in the usage documentation.
