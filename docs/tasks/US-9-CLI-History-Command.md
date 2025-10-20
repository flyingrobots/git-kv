# Task: CLI `history` Command

**Feature:** `US-9 — History across epochs`

## 1. Description

Implement the `git kv history <key>` command. This command provides a chronological log of all changes made to a specific key, traversing epoch boundaries if necessary.

## 2. Acceptance Criteria

- A `git kv history <key>` command is added to the CLI.
- It accepts a single argument: the `<key>` for which to retrieve history.
- The command outputs a human-readable log, one line per commit, in the format: `<abbrev_hash> <ISO_8601_UTC_date> <author> <commit_message> (Epoch: <epoch_id>, Op: <op_type>)`, where `<author>` is rendered exactly as `Author Name <author@example.com>` and `<op_type>` is one of `create`, `modify`, or `delete`.
  - **Example (modify):** `a1b2c3d4 2025-10-26T10:30:00Z John Doe <john@example.com> feat: Added new feature flag (Epoch: 2025Q4, Op: modify)`
  - **Example (delete):** `deadbeef 2025-11-02T09:00:00Z Jane Smith <jane@example.com> chore: remove deprecated flag (Epoch: 2025Q4, Op: delete)`
- A machine-readable JSON output option is available via a `--json` flag.
- "Relevant" commits are precisely defined as those that directly modified the specified key (create, modify, delete). This includes creation and deletion events.
- The command does not include unrelated directory-parent changes unless a `--recursive` or `--parent` flag is provided (future enhancement).
- If renames are supported, the command follows rename metadata to continue history across renames.
- **Epoch Traversal/Merge Semantics:** Commits from all epochs are shown, ordered by commit timestamp (newest first). Entries from merged branches are included (no deduplication).
- Colorization is controlled by a `--color` flag, not by default.

## 3. Test Plan

- **Integration Test (Chronological Order & Format):**
  - Set a key, then update it multiple times across different epochs.
  - Run `git kv history <key>`.
  - Assert that entries are ordered newest-first (most recent entry first).
  - Assert that each entry complies with the specified human-readable format (e.g., timestamp ISO8601, author, action, value).
- **Edge Case (Non-existent Key):**
  - Run `git kv history` for a key that does not exist.
  - Assert that the command returns "Key not found" on stdout and exits with code 0.
- **Edge Case (Deleted Key):**
  - Set a key, then delete it.
  - Run `git kv history <key>`.
  - Assert that the full history, including a specific deletion entry in the defined format, is shown in the correct chronological position.
