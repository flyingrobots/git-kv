# Task: CLI `remote setup` Command

**Feature:** `US-G12 — Discovery on Fresh Clone`

## 1. Description

Implement the `git kv remote setup` command. This command automates the configuration of the `origin.pushurl` for developers, pointing it to the Stargate based on the project's policy file.

## 2. Acceptance Criteria

- A `git kv remote setup` command is added to the CLI.
- It accepts an optional `--json` flag for machine-readable output.
- The command emits plain-text confirmation by default:
  - If `origin.pushurl` was previously unset: "Set remote.origin.pushurl to '<new_pushurl>'"
  - If `origin.pushurl` was updated: "Updated remote.origin.pushurl from '<old_pushurl>' to '<new_pushurl>'"
- When `--json` is used, it outputs a JSON object:
  ```json
  {
    "old_pushurl": "<old_pushurl> | null",
    "new_pushurl": "<new_pushurl>",
    "status": "set | updated"
  }
  ```
- The command exits 0 on success and nonzero on error, printing the error message to `stderr`.

## 3. Test Plan

- **Integration Test (Success):** In a fresh clone of a repository containing a valid policy file, run `git kv remote setup`. Verify with `git remote -v` that `origin.pushurl` is correctly set.
- **Integration Test (No Policy):** In a fresh clone without a `.kv/policy.yaml` file, run the command. Verify it fails with a clear error message.
- **Integration Test (Missing URL):** In a fresh clone with a policy file but no `stargate.push_url` defined, run the command. Verify it fails with a clear error message.
- **Idempotency:** Run the command multiple times and verify it consistently sets the correct `pushurl` without error.
