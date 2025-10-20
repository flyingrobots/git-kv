# Task: Stargate Policy Loader

**Feature:** `US-8 — Policy Enforcement`

## 1. Description

Implement the logic within the Stargate service to find, load, and parse the `.kv/policy.yaml` file from the repository. This parsed policy object will be used by the `pre-receive` hook to enforce rules.

## 2. Acceptance Criteria

- The Stargate service, on startup or on push, locates and reads the `.kv/policy.yaml` file from the repository's default branch.
- It uses a YAML parser to load the file content into a structured Go object.
- The Stargate service loads the policy at startup. On parse/load failure, it defaults to fail-safe (reject all pushes). This behavior can be overridden via the `KV_POLICY_FAILMODE` environment variable (values: "safe" or "open").
- The parsed policy object is injected into each `pre-receive` hook invocation as an explicit parameter (e.g., `runValidation(policy, push_event)`) to ensure a clear call contract.

## 3. Test Plan

- **Unit Test (Valid):** Verify YAML parsing with a valid policy file; check that the Go object is correctly populated.
- **Unit Test (Missing):** Verify behavior when the `.kv/policy.yaml` file is absent; ensure the configured fail-safe/fail-open strategy is applied.
- **Unit Test (Malformed):** Attempt to parse an invalid YAML file; verify the error is caught and handled per configuration.
- **Unit Test (Schema):** Parse a syntactically-valid YAML file with incorrect field names/types; verify schema validation catches this.
