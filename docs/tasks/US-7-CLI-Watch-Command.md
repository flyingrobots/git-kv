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
  - **Mechanism:** It detects new commits by polling the `refs/kv-watchlog/<ns>@<epoch>` ref on the remote.
  - **Polling Interval:** The polling interval defaults to 5 seconds and is configurable via a `--poll-interval` flag.
  - **State Tracking:** It tracks the last streamed commit OID to ensure only new, unseen changes are streamed.
  - **Latency:** The expected latency between a new change being published and the watch client seeing it is within `poll-interval + network_latency`.
  - **Termination:** The command runs indefinitely until manually terminated (e.g., Ctrl+C).

## 3. Test Plan

- **Integration Test:** Create several commits in the watchlog. Run `git kv watch` and verify the output matches the content of the commits.
- **Integration Test (`--since`):** Run `watch` with a `--since` OID from the middle of the log and verify only the later changes are printed.
- **Integration Test (`--follow`):** Run `watch -f`. In a separate process, create a new change. Verify the `watch` command prints the new change and does not exit.
