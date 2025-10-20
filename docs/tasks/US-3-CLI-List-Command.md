# Task: CLI `list` Command

**Feature:** `US-3 — Prefix list (index-only)`

## 1. Description

Implement the `git kv list` command, which provides an efficient way to find keys by prefix. This command must be optimized to use the `refs/kv-index/<ns>` branch and avoid a full scan of the main ledger.

## 2. Acceptance Criteria

- A `git kv list` command is added to the CLI.
- It accepts an optional `--prefix <p>` flag.
- If no prefix is given, it should list all keys (up to a reasonable limit or via pagination if implemented later).
- The command logic specifically targets the `refs/kv-index/<ns>` branch for its data.
- The output is a simple list of key names, one per line.

## 3. Test Plan

- **Integration Test:** Run `git kv list --help` and verify the output is correct.
- **Integration Test:** Run `git kv list` with no prefix and verify it lists all known keys.
- **Unit Test:** Test the command parsing logic to ensure the `--prefix` flag is handled correctly.
