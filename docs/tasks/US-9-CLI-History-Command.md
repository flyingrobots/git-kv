# Task: CLI `history` Command

**Feature:** `US-9 — History across epochs`

## 1. Description

Implement the `git kv history <key>` command. This command provides a chronological log of all changes made to a specific key, traversing epoch boundaries if necessary.

## 2. Acceptance Criteria

- A `git kv history <key>` command is added to the CLI.
- It accepts a single argument: the `<key>` for which to retrieve history.
- The command outputs a human-readable log, one line per commit, in the format: `<abbrev_hash> <ISO_8601_UTC_date> <author_name> <author_email> <commit_message> (Epoch: <epoch_id>, Op: <op_type>)`.
  - **Example:** `a1b2c3d4 2025-10-26T10:30:00Z John Doe <john@example.com> feat: Added new feature flag (Epoch: 2025Q4, Op: set)`
- A machine-readable JSON output option is available via a `--json` flag.
- "Relevant" commits are precisely defined as those that directly modified the specified key (create, modify, delete). This includes creation and deletion events.
- The command does not include unrelated directory-parent changes unless a `--recursive` or `--parent` flag is provided (future enhancement).
- If renames are supported, the command follows rename metadata to continue history across renames.
- **Epoch Traversal/Merge Semantics:** Commits from all epochs are shown, ordered by commit timestamp (newest first). Entries from merged branches are included (no deduplication).
- Colorization is controlled by a `--color` flag, not by default.

## 3. Test Plan

- **Integration Test:** Set a key, then update it multiple times. Run `git kv history <key>` and verify all changes are shown in the correct order.
- **Edge Case:** Run `git kv history` for a non-existent key and verify it returns an empty log or a clear message indicating the key was not found.
- **Edge Case:** Run `git kv history` for a key that was deleted and verify its full history, including the deletion, is shown.
