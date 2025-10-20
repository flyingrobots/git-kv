# Task: Integration Test for `mset` Transaction

**Feature:** `US-G2 — Atomic transaction via Stargate`

## 1. Description

Create a high-level integration test that validates the entire `mset` workflow from the client's perspective. This test will involve a real Git repository and a mock Stargate server to simulate the end-to-end process.

## 2. Acceptance Criteria

- The test sets up a bare repository to act as the Stargate.
- The test Stargate is configured with the `pre-receive` hook and validation logic.
- The test initializes a client-side repository and configures its `pushurl` to point to the test Stargate.
- **Success Scenario:** The test runs `git kv mset` with a valid batch file and asserts that the command succeeds and the Stargate repository is updated correctly.
- **Failure Scenario:** The test runs `git kv mset` with a deliberately invalid commit (e.g., bad attestation) and asserts that the command fails with the expected error message.
- The test cleans up all temporary repositories and mock servers after execution.

## 3. Test Plan

- **Execution:** The integration test is added to the project's test suite and is run as part of `make test` or a separate `make integration-test` command.
- **Validation:** Run the test and verify that both the success and failure scenarios pass their assertions.
