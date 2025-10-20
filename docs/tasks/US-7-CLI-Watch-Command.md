# Task: CLI `watch` Command

**Feature:** `US-7 — Watch Feed`

## 1. Description

Implement the `git kv watch` command, which allows a client to stream the change feed from the watchlog. This can be used to trigger downstream processes and services.

## 2. Acceptance Criteria

- A `git kv watch` command is added to the CLI.
- It accepts an optional `--since <oid>` flag, where `<oid>` is a commit hash from the watchlog.
- The command fetches the `refs/kv-watchlog/<ns>@<epoch>` ref from the remote.
- It then performs a `git log` on that ref.
- If `--since` is provided, the log starts from the specified OID; otherwise, it shows the entire log.
- The command prints the content of each commit blob in the log to stdout.
- The command can have a `--follow` or `-f` mode to wait for and stream new changes as they appear.

## 3. Test Plan

- **Integration Test:** Create several commits in the watchlog. Run `git kv watch` and verify the output matches the content of the commits.
- **Integration Test (`--since`):** Run `watch` with a `--since` OID from the middle of the log and verify only the later changes are printed.
- **Integration Test (`--follow`):** Run `watch -f`. In a separate process, create a new change. Verify the `watch` command prints the new change and does not exit.
