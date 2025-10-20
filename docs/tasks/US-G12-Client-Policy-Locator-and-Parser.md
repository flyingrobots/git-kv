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

- **Unit Test (Success):** Provide a mock repository with a valid policy file. Verify the function correctly parses the file and extracts the `push_url`.
- **Unit Test (Missing File):** Provide a mock repository without the policy file. Verify the function returns an appropriate error.
- **Unit Test (Malformed YAML):** Provide a mock policy file with invalid YAML. Verify the function returns a parsing error.
- **Unit Test (Missing Field):** Provide a mock policy file that is valid YAML but lacks the `stargate.push_url` field. Verify the function returns an error indicating the field is missing.
