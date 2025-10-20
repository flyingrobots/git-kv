# Task: Client Policy Locator and Parser

**Feature:** `US-G12 — Discovery on Fresh Clone`

## 1. Description

Implement the client-side logic to locate and parse the `.kv/policy.yaml` file. This module will be used by `git kv remote setup` and potentially other client commands that need to read policy information.

## 2. Acceptance Criteria

- A function `LoadPolicy(repoRoot string) (*Policy, error)` is defined.
- **Repository Root Resolution:** `repoRoot` is defined as the nearest ancestor directory containing a `.git` directory, unless an explicit `--root` or `--path` flag is provided to the CLI.
- The function searches for `.kv/policy.yaml` within the `repoRoot`.
- If found, it reads and parses the YAML content into a `Policy` Go struct.
- **Policy YAML Schema:**
  ```yaml
  version: 1 # Required, integer
  stargate: # Required object
    push_url: "string" # Required
    read_url: "string" # Optional
    require_mirror_ack_by_default: "boolean" # Optional, default false
    admins: # Optional, array of strings (SSH/GPG keys)
      - "ssh-ed25519 AAAAC3... admin1"
  namespaces: # Required object
    main: # Required object, namespace name
      ledger_mode: "string" # Required, e.g., "linear"
      require_signed_commits: "boolean" # Optional, default false
      max_value_inline: "string" # Optional, e.g., "1MiB"
      max_value_total: "string" # Optional, e.g., "100MiB"
      chunking: # Optional object
        mode: "string" # e.g., "fastcdc"
        min: "string" # e.g., "64KiB"
        avg: "string" # e.g., "256KiB"
        max: "string" # e.g., "1MiB"
      allowed_prefixes: # Optional, array of strings
        - "cfg:"
      writers: # Optional, array of strings (SSH/GPG keys)
        - "ssh-ed25519 AAAAC3... writer1"
      forbidden: # Optional, array of objects
        - lfs: "boolean" # e.g., {lfs: true}
      retention: # Optional object
        ttl_default: "string" # e.g., "365d"
        enforce_readside: "boolean" # Optional, default true
  ```
- **Go Struct Definition:**
  ```go
  type Policy struct {
      Version    int               `yaml:"version"`
      Stargate   StargatePolicy    `yaml:"stargate"`
      Namespaces map[string]NamespacePolicy `yaml:"namespaces"`
  }
  // ... (nested structs for StargatePolicy, NamespacePolicy, etc. with yaml tags)
  ```
- The function handles cases where the file is missing, malformed, or required fields are absent, returning clear, specific errors (e.g., "policy file not found", "malformed YAML", "stargate.push_url is required").

## 3. Test Plan

Fixtures for these tests are committed under `docs/tasks/testdata/policy_parser_fixtures/`.

- **Unit Test (Success):**
  - **Fixture:** `docs/tasks/testdata/policy_parser_fixtures/valid-policy.yaml`
    ```yaml
    version: 1
    stargate:
      push_url: "ssh://git@stargate.local/org/repo.git"
    ```
  - **Expected Output:** `stargate.push_url` value is `"ssh://git@stargate.local/org/repo.git"`.
  - Verify the function correctly parses the file and extracts the `push_url`.
- **Unit Test (Missing File):**
  - **Fixture:** An empty directory (no policy file).
  - **Expected Error:** A specific error string indicating the file was not found (e.g., "policy file not found").
  - Verify the function returns the expected error.
- **Unit Test (Malformed YAML):**
  - **Fixture:** `docs/tasks/testdata/policy_parser_fixtures/malformed-policy.yaml`
    ```yaml
    version: 1
    stargate:
      push_url: "ssh://git@stargate.local/org/repo.git"
    - This is not valid YAML
    ```
  - **Expected Error:** A specific YAML parsing error string.
  - Verify the function returns the expected parsing error.
- **Unit Test (Missing Field):**
  - **Fixture:** `docs/tasks/testdata/policy_parser_fixtures/missing-push-url.yaml`
    ```yaml
    version: 1
    stargate: {}
    ```
  - **Expected Error:** A specific error string indicating the `stargate.push_url` field is missing (e.g., "stargate.push_url not found in policy").
  - Verify the function returns the expected error.
