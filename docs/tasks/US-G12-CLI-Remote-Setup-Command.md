# Task: CLI `remote setup` Command

**Feature:** `US-G12 — Discovery on Fresh Clone`

## 1. Description

Implement the `git kv remote setup` command. This command automates the configuration of the `origin.pushurl` for developers, pointing it to the Stargate based on the project's policy file.

## 2. Acceptance Criteria

- A `git kv remote setup` command is added to the CLI.
- The command does not require any arguments.
- It automatically locates and reads the `.kv/policy.yaml` file from the repository root.
- It extracts the `stargate.push_url` value from the policy.
- It executes `git config remote.origin.pushurl <url>` to set the push URL.
- The command provides clear output to the user, confirming the action taken and the new `pushurl`.

## 3. Test Plan

- **Integration Test (Success):** In a fresh clone of a repository containing a valid policy file, run `git kv remote setup`. Verify with `git remote -v` that `origin.pushurl` is correctly set.
- **Integration Test (No Policy):** In a fresh clone without a `.kv/policy.yaml` file, run the command. Verify it fails with a clear error message.
- **Integration Test (Missing URL):** In a fresh clone with a policy file but no `stargate.push_url` defined, run the command. Verify it fails with a clear error message.
- **Idempotency:** Run the command multiple times and verify it consistently sets the correct `pushurl` without error.
