# Task: CLI `list` Command

**Feature:** `US-3 — Prefix list (index-only)`

## 1. Description

Implement the `git kv list` command, which provides an efficient way to find keys by prefix. This command must be optimized to use the `refs/kv-index/<ns>` branch and avoid a full scan of the main ledger.

## 2. Acceptance Criteria

- A `git kv list` command is added to the CLI.
- It accepts an optional `--prefix <p>` flag and returns keys whose names begin with the provided prefix.
- If no prefix is given, it lists every key available in the namespace index (streamed in lexicographic order) with no artificial limit.
- Results are ordered lexicographically by key name.
- When no keys match the specified prefix, exit with status `0` and emit no lines (tests may also accept a single line `(no keys)` if a `--show-empty` flag is added later; default behavior is silent).
- The command requires a namespace argument `--namespace <ns>` (short `-n <ns>`). When omitted, it must fail with a usage error unless a repository-level default namespace is configured (document how to detect this). The namespace is substituted into the branch `refs/kv-index/<ns>` (example: `--namespace prod-config` targets `refs/kv-index/prod-config`).
- If the `refs/kv-index/<ns>` branch is missing or unreadable, exit with non-zero status and print an error message explaining the branch is unavailable.
- Accept prefixes containing URL-safe characters (`[A-Za-z0-9._-]`). Reject other characters with a validation error. Future escaping rules must be documented if support expands.
- Output key names one per line using UTF-8; control characters (newline, tab, carriage return) must be escaped using JSON-style escapes (e.g., `\n`, `\t`, `\r`).
- The command logic specifically targets the `refs/kv-index/<ns>` branch for its data (e.g., `git kv list -n prod-config --prefix foo/` reads from `refs/kv-index/prod-config`).

## 3. Test Plan

- **Integration Test:** Run `git kv list --help` and verify the output is correct.
- **Integration Test:** Seed the index with a modest number of keys (<1000); run `git kv list` with no prefix and verify it returns every seeded key in lexicographic order with no truncation.
- **Integration Test:** Seed the index with more than 1000 keys; run `git kv list` with no options and confirm all entries are streamed (tests may simulate by checking the first, Nth, and last key rather than loading everything into memory).
- **Integration Test:** Run `git kv list --prefix non-existent` on a populated index and confirm exit code `0` with empty stdout.
- **Integration Test:** Remove or rename `refs/kv-index/<ns>` and invoke the command; assert non-zero exit and error message mentioning the missing branch.
- **Integration Test:** Omit `--namespace` and ensure the command exits with usage guidance (unless repository default is configured, in which case verify the default namespace branch is used).
- **Unit Test:** Verify prefixes containing only URL-safe characters are accepted and others are rejected with validation errors.
- **Unit Test:** Provide keys with control characters and ensure output escapes them as `\n`, `\t`, or `\r`.
- **Unit Test:** Test the command parsing logic to ensure the `--prefix` flag is handled correctly.
