# Task: CLI `watch` Command

**Feature:** `US-7 — Watch Feed`

## 1. Description

Implement the `git kv watch` command, which allows a client to stream the change feed from the watchlog. This can be used to trigger downstream processes and services.

## 2. Acceptance Criteria

- A `git kv watch` command is added to the CLI.
- It accepts an optional `--since <oid>` flag, where `<oid>` is a commit hash from the watchlog. If the OID cannot be found in the watchlog, the CLI must print `Error: specified OID <oid> not found in watchlog` and exit with code `3`.
- The command fetches the `refs/kv-watchlog/<ns>@<epoch>` ref from the remote. If the ref is absent, it must exit with code `2` and message `Error: remote ref refs/kv-watchlog/<ns>@<epoch> not found`.
- It then performs a `git log` on that ref. Remote connectivity failures must surface as `Error: failed to contact remote <remote-name> — <cause>` with exit code `1`; the CLI does not perform automatic retries.
- If `--since` is provided, the log starts from the specified OID; otherwise, it streams only the most recent `N` commits (default `N=1000`). Users may override with `--limit <n>` (new flag) up to hard maximum `10000`; requests beyond the maximum must fail fast with a descriptive error requiring `--since` or a smaller `--limit`.
- The command prints the content of each commit blob in the log to stdout, emitting data in streaming chunks to avoid excessive memory usage.
- The command can have a `--follow` or `-f` mode to wait for and stream new changes as they appear.
  - **Mechanism:** It detects new commits by polling the `refs/kv-watchlog/<ns>@<epoch>` ref on the remote.
  - **Polling Interval:** The polling interval defaults to 5 seconds and is configurable via a `--poll-interval` flag.
  - **State Tracking:** Persist the last streamed OID per user/namespace in `~/.git-kv/watch/<namespace>.json` containing `last_streamed_oid`, `epoch`, and `timestamp`. Writes must use temp file + atomic rename and `fsync`. On startup, read and validate; if corrupt move to `.corrupt` and start fresh; if missing, start empty. Use an advisory lock to prevent concurrent runs; if locking fails, exit with error. When the state directory is unavailable, fall back to in-memory tracking with a warning.
  - **Latency:** After the initial sync completes, new changes must surface within `poll-interval + network_latency`. Initial fetch may add up to `2 * poll-interval` of additional delay.
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
- **Integration Test (Limit Guardrail):** Run `git kv watch --limit 20000` and assert it exits with code `4` and the limit error message.
- **Integration Test (Missing Ref):** Point CLI to a remote without `refs/kv-watchlog/...`; assert exit code `2` and error message.
- **Integration Test (Bad --since):** Provide an unknown OID; assert exit code `3` and message includes OID.
- **Integration Test (State Persistence):** Run watch command, terminate, ensure state file created with expected JSON; restart and assert it resumes after stored OID.
- **Unit Test (State Corruption):** Provide corrupt state file; assert CLI moves file to `.corrupt` and starts fresh.
- **Integration Test (Concurrent Run):** Start watch, then start second instance; assert second exits with locking error (code `5`).

## Errors and Exit Codes

- `1`: Remote unreachable — message `Error: failed to contact remote <remote-name> — <short cause>`.
- `2`: Required watchlog ref missing — message `Error: remote ref refs/kv-watchlog/<ns>@<epoch> not found`.
- `3`: `--since` OID not present in watchlog — message `Error: specified OID <oid> not found in watchlog`.
- `4`: Limit validation error (requested entries exceed hard cap) — message `Error: requested limit <n> exceeds maximum 10000; provide --since or smaller --limit`.
- `5`: State persistence error or lock acquisition failure — message `Error: unable to persist watch state (<details>)`.
- The CLI does not perform automatic retries for these errors; callers should implement their own retry logic where appropriate.
