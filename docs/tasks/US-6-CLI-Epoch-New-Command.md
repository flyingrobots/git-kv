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

- **Integration Test (Happy Path):** Run `git kv epoch new --label 2026Q1`. Verify `refs/kv-epoch/<ns>` advanced and `refs/kv-chunks/<ns>@2026Q1` created.
- **Failure Case (Duplicate Label):** Pre-create `refs/kv-chunks/<ns>@2026Q1`. Re-run command with same label; expect exit code `1`, error `Error: Epoch label 2026Q1 already exists`, and no ref changes.
- **Failure Case (Empty Label):** `git kv epoch new --label ""` should exit `2` with validation error.
- **Failure Case (Whitespace Handling):** Labels with leading/trailing spaces (" 2026Q1", "2026 Q1") should be rejected with validation error unless explicitly trimmed per spec; assert chosen behavior.
- **Failure Case (Invalid Characters):** Test labels containing `/`, `@`, `..`, or uppercase/lowercase as per allowed charset; ensure forbidden chars cause rejection with explicit message.
- **Failure Case (Length Boundary):** Attempt label at max allowed length (e.g., 64 chars) succeeds; exceeding limit fails with clear error.
- **Concurrency Test:** Run two simultaneous `epoch new` commands for same label; ensure only one succeeds and other gets deterministic duplicate error; verify refs remain consistent.
- **Partial Failure Simulation:** Mock failure after chunk ref creation but before epoch ref update; assert transaction rolls back (no dangling refs) and user receives guidance to retry.
- **Permissions Test:** Simulate lack of push rights; verify command surfaces permission denied and no state change.
