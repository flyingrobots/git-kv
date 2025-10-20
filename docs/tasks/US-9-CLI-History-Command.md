# Task: CLI `history` Command

**Feature:** `US-9 — History across epochs`

## 1. Description

Implement the `git kv history <key>` command. This command provides a chronological log of all changes made to a specific key, traversing epoch boundaries if necessary.

## 2. Acceptance Criteria

- A `git kv history <key>` command is added to the CLI.
- It accepts a single argument: the `<key>` for which to retrieve history.
- The command outputs a formatted log of commits, similar to `git log`, but filtered to show only changes relevant to the specified key.
- The output should clearly indicate the commit hash, author, date, and commit message for each change.

## 3. Test Plan

- **Integration Test:** Set a key, then update it multiple times. Run `git kv history <key>` and verify all changes are shown in the correct order.
- **Edge Case:** Run `git kv history` for a non-existent key and verify it returns an empty log or a clear message indicating the key was not found.
- **Edge Case:** Run `git kv history` for a key that was deleted and verify its full history, including the deletion, is shown.
