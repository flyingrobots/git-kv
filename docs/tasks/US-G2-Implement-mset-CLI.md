# Task: Implement `mset` CLI Command

**Feature:** `US-G2 — Atomic transaction via Stargate`

## 1. Description

Implement the basic command structure for `git kv mset` in the Go CLI application. This includes defining the command, its flags (e.g., `--file`), and help text. This task does not include the logic for processing the file, only the command-line interface itself.

## 2. Acceptance Criteria

- The `git-kv` binary responds to the `mset` subcommand.
- The `mset` command accepts a required `--file <path>` flag.
- Running `git kv mset --help` displays clear usage information for the command and its flags.
- If the `--file` flag is not provided, the command exits with an error and suggests the correct usage.

## 3. Test Plan

- **Unit Test:** Write a test that verifies the `mset` command is registered in the CLI application's command tree.
- **Integration Test:** Execute the compiled binary with `mset --help` and assert that the output contains the expected usage string.
- **Integration Test:** Execute the compiled binary with `mset` but without the `--file` flag and assert that it returns a non-zero exit code and an appropriate error message to stderr.
