# Task: CLI `epoch new` Command

**Feature:** `US-6 — Epoch Rollover`

## 1. Description

Implement the `git kv epoch new --label <name>` command. This is an administrative command used to create a new epoch, which effectively snapshots the database and allows new clones to be smaller.

## 2. Acceptance Criteria

- A `git kv epoch new` command is added to the CLI.
- It requires a `--label <name>` flag, which will be the name of the new epoch (e.g., `2026Q1`).
- The command creates a new, empty `refs/kv-chunks/<ns>@<new_label>` ref.
- It updates the `refs/kv-epoch/<ns>` ref to point to a commit that contains the new label.
- The command should only be runnable by users with appropriate permissions (if a permissions model is in place).

## 3. Test Plan

- **Integration Test:** Run `git kv epoch new --label 2026Q1`. Verify with `git show-ref` that the `refs/kv-epoch/main` and `refs/kv-chunks/main@2026Q1` refs have been created or updated correctly.
- **Failure Case:** Run the command without a label and verify it fails with a clear error message.
- **Failure Case:** Run the command with an invalid label format and verify it is rejected.
