# Task: CLI `stargate sync` Command

**Feature:** `US-G11 — Split-Brain Recovery`

## 1. Description

Implement the `git kv stargate sync` command. This command is an administrative tool used to compare and reconcile the state of the Stargate (source of truth) with its GitHub mirror, especially in recovery scenarios.

## 2. Acceptance Criteria

- A `git kv stargate sync` command is added to the CLI.
- It accepts an optional `--ns <ns>` flag to specify the namespace. When provided, only refs under that namespace are diffed/repaired; if the namespace does not exist the command prints `Error: namespace <ns> not found` and exits with code `3`. When omitted, all namespaces are processed.
- Example invocations:
  - `git kv stargate sync --ns prod --dry-run` → reports differences limited to `prod`.
  - `git kv stargate sync --dry-run` → examines every namespace.
  - `git kv stargate sync --ns missing --repair` → exits `3` with namespace-not-found error.
- It accepts mutually exclusive flags: `--dry-run`, `--repair`, and `--force`.
- If no flags are provided, it defaults to `--dry-run`.
- **`--dry-run` mode:**
  - Produces a human-readable report (summary counts) and a machine-readable JSON report (via `--output=json`) listing all detected differences without making any changes.
  - Exit code: `0` (no changes made).
- **`--repair` mode:**
  - Performs only idempotent, reversible fixes (e.g., fast-forward pushes for refs where Stargate is ahead, updates metadata).
  - Skips operations considered "unsafe" (e.g., non-fast-forward pushes).
  - Exit codes: `0` (mirror already aligned or repairs succeeded), `1` (some safe repairs failed but processing continued), `2` (fatal error aborted execution).
- **`--force` mode:**
  - Performs irreversible overwrites (e.g., `git push --force` to replace mirror content with Stargate state).
  - Requires admin authorization (checked against `stargate.admins` in `.kv/policy.yaml`).
  - Requires a confirmation prompt (unless `--yes` is provided).
  - All force operations are audited and logged.
  - Exit codes: `0` (force sync succeeded), `1` (authorization or confirmation failure), `2` (fatal error).
- Safe metadata updates in `--repair` mode include updating ref timestamps, adding or repairing lightweight tags, and correcting mirror tracking annotations; operations that rewrite history, change commit parents, or tamper with signed tags are forbidden and must be reported as unsafe.
- Exit code summary with examples: `0` (mirror already aligned or repairs completed; e.g., `--dry-run` on clean state), `1` (non-fatal issues; e.g., `--repair` skipped unsafe ref or `--force` authorization denied), `2` (fatal error aborting processing; e.g., network failure mid-push), `3` (`--ns` specified namespace not found).
- The command provides clear output (console summary, verbose logs, and optional `--output=json` file) indicating the status of the mirror and any proposed or executed actions.
- The policy file defaults to `.kv/policy.yaml` relative to the repository root; users may override via `--policy <path>` or `GIT_KV_POLICY`. Missing policy file results in exit code `2` with actionable error.

## 3. Test Plan

- **Unit Test:** Test the command-line parsing for all flags and their mutual exclusivity.
- **Integration Test (Dry Run):**
  - **Setup:** Create a test repository with a known divergence (Stargate ahead, mirror ahead, or both).
  - **Action:** Run `git kv stargate sync --dry-run`.
  - **Assert:** The output (console and `--output=json`) accurately lists all detected differences without applying any changes.
- **Integration Test (Repair Mode):**
  - **Setup:** Create a test repository where Stargate is ahead (fast-forwardable).
  - **Action:** Run `git kv stargate sync --repair`.
  - **Assert:** The mirror is successfully updated.
  - **Setup:** Create a test repository where the mirror is ahead (non-fast-forward).
  - **Action:** Run `git kv stargate sync --repair`.
  - **Assert:** The command reports a non-fast-forward error and does not push.
- **Integration Test (Force Mode - Authorization):**
  - **Setup:** Configure a test policy with an admin user.
  - **Action:** Run `git kv stargate sync --force` as a non-admin. Assert rejection.
  - **Action:** Run `git kv stargate sync --force` as an admin. Assert success (after confirmation).
- **Integration Test (Force Mode - Overwrite):**
  - **Setup:** Create a test repository with a known divergence (mirror has unique commits).
  - **Action:** Run `git kv stargate sync --force` as an admin.
  - **Assert:** The mirror's state is forcibly overwritten to match the Stargate.
- **Integration Test (Error Handling & Recovery):**
  - Simulate transient network errors during push. Assert retries and eventual success.
  - Simulate permanent network errors. Assert appropriate failure and logging.
- **Integration Test (Idempotency):** Run `git kv stargate sync --repair` or `--force` multiple times on a synchronized repo. Assert no changes are made after the first successful run.
- **Integration Test (`--ns` flag):** Create namespaces `alpha`, `beta`; diverge only `alpha`. Run `--ns alpha` and assert only alpha refs repaired. Run `--ns gamma` (non-existent) and assert exit code `3` with error message.
- **Unit Test (Policy Path Override):** Ensure `--policy` flag and `GIT_KV_POLICY` environment variable are honored; missing policy triggers exit code `2`.
- **Integration Test (Metadata Guardrails):** Attempt to rewrite signed tag via `--repair`; assert refusal with forbidden metadata error.
