# Task: Client Policy Locator and Parser

**Feature:** `US-G12 — Discovery on Fresh Clone`

## 1. Description

Implement the client-side logic to locate and parse the `.kv/policy.yaml` file. This module will be used by `git kv remote setup` and potentially other client commands that need to read policy information.

## 2. Acceptance Criteria

- A function `LoadPolicy(repoRoot string) (*Policy, error)` is defined.
- **Repository Root Resolution:** `repoRoot` is defined as the nearest ancestor directory containing a `.git` directory, unless an explicit `--root` or `--path` flag is provided to the CLI.
- The function resolves the policy path strictly to `repoRoot/.kv/policy.yaml` (no recursive search). `repoRoot` is either the explicit `--root` flag provided by the caller or, by default, the nearest ancestor of the requested path containing a `.git` directory (even if that ancestor is itself a worktree). Only that directory is checked; worktree parents are ignored. If the file is absent, return `POLICY_NOT_FOUND`.
- If found, it reads and parses the YAML content into a `Policy` Go struct.
- **Example:** `LoadPolicy("/repo/worktrees/wt-a")` checks only `/repo/worktrees/wt-a/.kv/policy.yaml`; it does not fall back to `/repo/.kv/policy.yaml`.
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
      forbidden: # Optional object mapping features to booleans
        lfs: true
        unmanaged_uploads: false
      retention: # Optional object
        ttl_default: "string" # e.g., "365d"
        enforce_readside: "boolean" # Optional, default true
      storage: # Optional object
        backend: "string"
        max_object_size: "string" # e.g., "500MiB"
      replication: # Optional object
        enabled: true
        targets: ["us-east-1", "us-west-2"]
        max_lag: "10m"
      ingestion: # Optional object
        allowed_formats: ["jsonl", "csv"]
        max_batch_size: "20MiB"
- **Size & Duration Formats:**
  - Sizes accept integers optionally followed by unit suffix: `B`, `KB`, `MB`, `GB` (decimal) or `KiB`, `MiB`, `GiB` (binary). Decimal units use powers of 1000; binary units use powers of 1024. Units are case-sensitive and must match exactly; whitespace is not permitted. Floating point values are rejected. Parsed values are stored as `uint64` bytes.
  - Durations accept integers with suffix `s`, `m`, `h`, `d`, or `w` (seconds, minutes, hours, days, weeks). Units are case-sensitive; mixed or floating values are rejected. Parsed values are stored as `time.Duration` (nanoseconds) or `int64` seconds.
  - Invalid formats emit validation errors specifying the offending value; overflow beyond `uint64` or `time.Duration` maximums also fails validation.
  ```
- **Go Struct Definition:**
  ```go
  type SizeBytes uint64

  type DurationSeconds int64

  // SizeBytes and DurationSeconds must implement yaml.Unmarshaler to enforce parsing rules.

  type Policy struct {
      Version    int                           `yaml:"version"`
      Stargate   StargatePolicy                `yaml:"stargate"`
      Namespaces map[string]NamespacePolicy    `yaml:"namespaces"`
  }

  type StargatePolicy struct {
      PushURL                 string   `yaml:"push_url"`
      ReadURL                 string   `yaml:"read_url,omitempty"`
      RequireMirrorAckByDefault bool    `yaml:"require_mirror_ack_by_default"`
      Admins                  []string `yaml:"admins,omitempty"`
  }

  type NamespacePolicy struct {
      LedgerMode           string            `yaml:"ledger_mode"`
      RequireSignedCommits bool              `yaml:"require_signed_commits"`
      MaxValueInline       *SizeBytes        `yaml:"max_value_inline,omitempty"`
      MaxValueTotal        *SizeBytes        `yaml:"max_value_total,omitempty"`
      Chunking             *ChunkingPolicy   `yaml:"chunking,omitempty"`
      AllowedPrefixes      []string          `yaml:"allowed_prefixes,omitempty"`
      Writers              []string          `yaml:"writers,omitempty"`
      Forbidden            map[string]bool   `yaml:"forbidden,omitempty"`
      Retention            *RetentionPolicy  `yaml:"retention,omitempty"`
      Storage              *StoragePolicy    `yaml:"storage,omitempty"`
      Replication          *ReplicationPolicy `yaml:"replication,omitempty"`
      Ingestion            *IngestionPolicy  `yaml:"ingestion,omitempty"`
  }

  type ChunkingPolicy struct {
      Mode string      `yaml:"mode"`
      Min  SizeBytes   `yaml:"min"`
      Avg  SizeBytes   `yaml:"avg"`
      Max  SizeBytes   `yaml:"max"`
  }

  type RetentionPolicy struct {
      TTLDefault      DurationSeconds `yaml:"ttl_default"`
      EnforceReadside *bool           `yaml:"enforce_readside,omitempty"`
  }

  type StoragePolicy struct {
      Backend        string     `yaml:"backend"`
      MaxObjectSize  *SizeBytes `yaml:"max_object_size,omitempty"`
      Encryption     *bool      `yaml:"encryption,omitempty"`
  }

  type ReplicationPolicy struct {
      Enabled          bool             `yaml:"enabled"`
      Targets          []string         `yaml:"targets,omitempty"`
      MaxLag           *DurationSeconds `yaml:"max_lag,omitempty"`
  }

  type IngestionPolicy struct {
      AllowedFormats []string `yaml:"allowed_formats,omitempty"`
      MaxBatchSize   *SizeBytes `yaml:"max_batch_size,omitempty"`
  }
  ```
- The function handles cases where the file is missing, malformed, or required fields are absent, returning clear, specific errors (e.g., "policy file not found", "malformed YAML", "stargate.push_url is required"). Error responses must include a machine code (e.g., `POLICY_NOT_FOUND`, `POLICY_INVALID_FIELD`) and human-readable message.
- Errors are returned as `PolicyError` values with fields `Code string`, `Message string`, and optional `Details map[string]any`.

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
- **Unit Test (Nested Policy Ignored):** Provide a policy file under a subdirectory; assert loader ignores it when `repoRoot/.kv/policy.yaml` is absent.
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
- **Unit Test (Size Parsing):** Provide policies with values like `64KiB`, `1MB`, `1.5MB`. Assert valid ones parse to correct `SizeBytes` and invalid (floats, unknown units) return validation errors.
- **Unit Test (Duration Parsing):** Provide policies with `30s`, `10m`, `2h`, `365d`, `1w`. Assert correct parsing and that invalid inputs (e.g., `1.5h`, `30seconds`) fail.
