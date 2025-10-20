# Task: CLI `stargate sync` Command

**Feature:** `US-G11 — Split-Brain Recovery`

## 1. Description

Implement the `git kv stargate sync` command. This command is an administrative tool used to compare and reconcile the state of the Stargate (source of truth) with its GitHub mirror, especially in recovery scenarios.

## 2. Acceptance Criteria

- A `git kv stargate sync` command is added to the CLI.
- It accepts an optional `--ns <ns>` flag to specify the namespace.
- It accepts mutually exclusive flags: `--dry-run`, `--repair`, and `--force`.
- If no flags are provided, it defaults to `--dry-run`.
- The command provides clear output indicating the status of the mirror and any proposed or executed actions.

## 3. Test Plan

- **Unit Test:** Test the command-line parsing for all flags and their mutual exclusivity.
- **Integration Test:** Run `git kv stargate sync --help` and verify the output is correct.
