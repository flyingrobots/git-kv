# Task: Client Policy Locator and Parser

**Feature:** `US-G12 — Discovery on Fresh Clone`

## 1. Description

Implement the client-side logic to locate and parse the `.kv/policy.yaml` file. This module will be used by `git kv remote setup` and potentially other client commands that need to read policy information.

## 2. Acceptance Criteria

- A function exists that searches for `.kv/policy.yaml` in the repository root.
- If found, it reads and parses the YAML content into a structured Go object.
- The function can safely extract specific fields, such as `stargate.push_url`.
- It handles cases where the file is missing, malformed, or the requested field is absent, returning clear errors.

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
