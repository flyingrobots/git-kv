# Task: CLI `list` Command

**Feature:** `US-3 — Prefix list (index-only)`

## 1. Description

Implement the `git kv list` command, which provides an efficient way to find keys by prefix. This command must be optimized to use the `refs/kv-index/<ns>` branch and avoid a full scan of the main ledger.

## 2. Acceptance Criteria

- A `git kv list` command is added to the CLI.
- It accepts an optional `--prefix <p>` flag and returns keys whose names begin with the provided prefix.
- It accepts an optional `--limit <n>` flag (default `1000`). After filtering keys by the prefix (or no prefix), the command returns at most `n` matching keys. Pagination beyond this limit remains a future enhancement.
- If no prefix is given, it still applies the default limit of 1000 keys (or the caller-provided `--limit`) while streaming results in lexicographic order.
- Results are ordered lexicographically by key name.
- When no keys match the specified prefix, exit with status `0` and emit no lines (tests may also accept a single line `(no keys)` if a `--show-empty` flag is added later; default behavior is silent).
- The command requires a namespace argument `--namespace <ns>` (short `-n <ns>`). If omitted, the CLI MUST read `git config kv.defaultNamespace`; when set, that value is used as the namespace, otherwise exit non-zero with the usage error `--namespace is required; run 'git config kv.defaultNamespace <ns>' to set a default.` The namespace (from flag or config) is substituted into the branch `refs/kv-index/<ns>` (example: `--namespace prod-config` targets `refs/kv-index/prod-config`).
- If the `refs/kv-index/<ns>` branch is missing or unreadable, exit with non-zero status and print an error message explaining the branch is unavailable.
- Validate `--limit` as a positive integer; zero, negative, or non-integer values must trigger a parsing error with non-zero exit.
- Accept prefixes containing URL-safe characters (`[A-Za-z0-9._-]`). Reject other characters with a validation error. Future escaping rules must be documented if support expands.
- Output key names one per line using UTF-8; control characters (newline, tab, carriage return) must be escaped using JSON-style escapes (e.g., `\n`, `\t`, `\r`).
- The command logic specifically targets the `refs/kv-index/<ns>` branch for its data (e.g., `git kv list -n prod-config --prefix foo/` reads from `refs/kv-index/prod-config`).

## 3. Test Plan

- **Integration Test:** Run `git kv list --help` and verify the output is correct.
- **Integration Test:** Seed the index with a small dataset (e.g., 50 keys); run `git kv list` with no prefix and verify it returns every seeded key in lexicographic order.
- **Integration Test:** Seed the index with a larger dataset (use ~1,000 keys as a representative load test—this is a test scale, not a protocol limit); run `git kv list` with no options and verify it returns 1,000 entries (default limit) and that the last entry matches the expected lexicographic boundary. Then run with `--limit 1` to confirm truncation and `--limit 1500` to confirm the higher ceiling returns additional keys when available.
- **Integration Test:** Seed distinct prefixes, run `git kv list --prefix foo/ --limit 10`, and verify it returns at most 10 keys whose names start with `foo/`.
- **Integration Test:** Run `git kv list --prefix non-existent` on a populated index and confirm exit code `0` with empty stdout.
- **Integration Test:** Remove or rename `refs/kv-index/<ns>` and invoke the command; assert non-zero exit and error message mentioning the missing branch.
- **Integration Test:** Omit `--namespace` with no default configured and assert the command exits non-zero with the exact usage error text. Then set `git config kv.defaultNamespace staging` and rerun to verify the command targets `refs/kv-index/staging` without requiring the flag.
- **Unit Test:** Validate `--limit` parsing rejects zero, negative, and non-integer values with informative errors.
- **Unit Test:** Verify prefixes containing only URL-safe characters are accepted and others are rejected with validation errors.
- **Unit Test:** Provide keys with control characters and ensure output escapes them as `\n`, `\t`, or `\r`.
- **Unit Test:** Test the command parsing logic to ensure the `--prefix` flag is handled correctly.
