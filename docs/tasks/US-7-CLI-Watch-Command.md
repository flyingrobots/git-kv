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
- **Integration Test (`--follow` mode):**
  1. Start `git kv watch -f --poll-interval 2s` in a background process.
  2. Wait 3 seconds.
  3. In a separate process, create a new change (e.g., `git kv set`).
  4. Within 5 seconds, verify that the `watch` command prints the new change.
  5. Verify the `watch` command continues to run and does not exit.
  6. Implement cleanup procedures to reliably terminate the background `watch` process.
- **Integration Test (Latency):**
  1. Start `git kv watch -f --poll-interval 1s` in a background process.
  2. Record timestamp T1.
  3. In a separate process, create a new change.
  4. Record timestamp T2 when the `watch` command prints the change.
  5. Assert that `T2 - T1` is within an acceptable latency bound (e.g., < 3 seconds).
