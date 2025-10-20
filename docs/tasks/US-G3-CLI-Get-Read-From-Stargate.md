# Task: CLI `get --read-from=stargate`

**Feature:** `US-G3 — Mirror Resilience & Read-After-Write`

## 1. Description

Modify the existing `git kv get` command to support a `--read-from=stargate` flag. This provides an alternative Read-Your-Writes (RYW) mechanism by allowing a client to bypass the potentially stale GitHub mirror and read directly from the source of truth.

## 2. Acceptance Criteria

- The `git kv get` command accepts a `--read-from` flag which can take the value `stargate` (or `origin` for the default).
- When `--read-from=stargate` is used, the command logic fetches the requested key from the Git remote defined in `remote.origin.pushurl`.
- When the flag is not used, it defaults to reading from the standard `remote.origin.url`.
- If `--read-from=stargate` is used and `remote.origin.pushurl` is not configured, the command **must** fail with exit code `PUSHURL_NOT_FOUND` and print the exact message: "Error: remote.origin.pushurl is not configured; use 'git remote set-url --push' to set it."

## 3. Test Plan

- **Integration Test:** Set up a test repo with a `url` pointing to a mock mirror and a `pushurl` pointing to a mock Stargate. Create a key and push it to the Stargate. Immediately run `get --read-from=stargate` and verify it returns the new value. Run a standard `get` and verify it returns an old or non-existent value (simulating mirror lag).
- **Unit Test:** Test the logic that selects the correct remote URL based on the flag.
- **Failure Case:** Run the command with `--read-from=stargate` in a repo where `remote.origin.pushurl` is not configured. Assert that the command exits with code `PUSHURL_NOT_FOUND` and prints the exact error message: "Error: remote.origin.pushurl is not configured; use 'git remote set-url --push' to set it."
