# Task: Stargate Pre-receive Hook Script

**Feature:** `US-G2 — Atomic transaction via Stargate`

## 1. Description

Create the `pre-receive` hook script for the Stargate server. This script is the entry point for all validation. It reads the details of the incoming push from stdin and orchestrates the various validation checks, including the crucial O(1) attestation validation.

## 2. Acceptance Criteria

- A `pre-receive` executable script is installed in the Stargate's bare repository `hooks` directory.
- The script reads the `old`, `new`, and `ref` values for all updated refs from stdin.
- It identifies pushes that contain `refs/kv/` and `refs/kv-index/` updates.
- It invokes a separate, more powerful executable (e.g., the main `git-kv-stargate` Go binary) to perform the actual validation logic, passing the ref information to it.
- The script exits 0 if the validation binary returns success.
- The script exits with a non-zero status and prints the error message to stderr if the validation binary returns failure.

## 3. Test Plan

- **Integration Test:** Push a valid commit to a test repository with the hook installed. Verify the push is accepted.
- **Integration Test:** Push an invalid commit. Verify the push is rejected and that the error message from the validation logic is correctly printed to the client.
- **Edge Case:** Test a standard `git push` of a non-`git-kv` branch (e.g., `main`) and ensure the hook does not interfere with it.
