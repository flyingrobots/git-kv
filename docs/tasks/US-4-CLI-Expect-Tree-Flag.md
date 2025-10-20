# Task: CLI `--expect-tree` Flag

**Feature:** `US-4 — CAS on branch`

## 1. Description

Add the `--expect-tree <OID>` flag to the `git kv set` and `git kv mset` commands. This flag is the entry point for the user to initiate a Compare-And-Swap operation.

## 2. Acceptance Criteria

- The `set` and `mset` commands accept a new string flag `--expect-tree <oid>` (short `-E <oid>` if available) and pass the value through to the client library.
- The CLI validates the OID before invoking Git: it must match `^[0-9a-f]{40}$` (SHA-1) or `^[0-9a-f]{64}$` (SHA-256). On failure, exit with code `2` and print `Invalid OID: expected 40- or 64-character hexadecimal SHA; got '<value>'` to stderr.
- The client library defensively re-validates the OID and throws a typed `ValidationError` (or equivalent) with the same message so callers receive consistent feedback if validation is bypassed.

## 3. Test Plan

- **Unit Test (Parsing):** Ensure both `git kv set` and `git kv mset` register the `--expect-tree/-E` option and forward a valid value to the application layer.
- **Integration Test (Valid OID passthrough):** Invoke `git kv set --expect-tree 0123456789abcdef0123456789abcdef01234567` and confirm the CLI exits `0`, passes the exact OID to the client stub, and does not emit validation errors.
- **Integration Test (Invalid OIDs):**
  - `git kv set --expect-tree deadbeef` → exit code `2`, stderr contains `Invalid OID...deadbeef`.
  - `git kv set --expect-tree zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz` → exit code `2`, stderr validation message.
  - `git kv set --expect-tree 0123456789abcdef0123456789abcdef0123456789abcdef` (wrong length 48) → exit code `2`, same error format.
- **Integration Test (Missing value):** `git kv set --expect-tree` should fail parsing, exit `2`, and print a helpful usage error indicating the flag requires a value.
- **Integration Test (mset parity):** Repeat the valid/invalid cases above with `git kv mset` to confirm symmetric behavior and that conflicting `--expect-tree` options across subcommands surface identical validation.
- **Integration Test (Help text):** `git kv set --help` output includes the flag name, description "Compare-And-Swap entry point", notes that the value is an OID (40/64 hex), and shows a usage example (e.g., `--expect-tree 0123...`).
