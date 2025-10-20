# Task: CLI `stargate sync` Command

**Feature:** `US-G11 — Split-Brain Recovery`

## 1. Description

Implement the `git kv stargate sync` command. This command is an administrative tool used to compare and reconcile the state of the Stargate (source of truth) with its GitHub mirror, especially in recovery scenarios.

## 2. Acceptance Criteria

- A `git kv stargate sync` command is added to the CLI.
- It accepts an optional `--ns <ns>` flag to specify the namespace.
- It accepts mutually exclusive flags: `--dry-run`, `--repair`, and `--force`.
- If no flags are provided, it defaults to `--dry-run`.
- **`--dry-run` mode:**
  - Produces a human-readable report (summary counts) and a machine-readable JSON report (via `--output=json`) listing all detected differences without making any changes.
  - Exit code: `0` (no changes made).
- **`--repair` mode:**
  - Performs only idempotent, reversible fixes (e.g., fast-forward pushes for refs where Stargate is ahead, updates metadata).
  - Skips operations considered "unsafe" (e.g., non-fast-forward pushes).
  - Exit code: `0` (all good/no repairs), `1` (non-fatal repair failures but completed processing), `2` (fatal error that stops processing).
- **`--force` mode:**
  - Performs irreversible overwrites (e.g., `git push --force` to replace mirror content with Stargate state).
  - Requires admin authorization (checked against `stargate.admins` in `.kv/policy.yaml`).
  - Requires a confirmation prompt (unless `--yes` is provided).
  - All force operations are audited and logged.
  - Exit code: `0` (success), `1` (authorization failure), `2` (fatal error).
- The command provides clear output (console summary, verbose logs, and optional `--output=json` file) indicating the status of the mirror and any proposed or executed actions.

## 3. Test Plan

- **Unit Test:** Test the command-line parsing for all flags and their mutual exclusivity.
- **Integration Test:** Run `git kv stargate sync --help` and verify the output is correct.
