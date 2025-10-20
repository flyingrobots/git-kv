# Task: Implement CI Test Job

**Feature:** `Foundational CI/CD Pipeline`

## 1. Description

Add a job to the CI workflow that checks out the code, sets up the Go environment, and runs the project's tests. Ensure the Go modules cache remains enabled so repeated runs reuse dependencies.

**Note on Action Versions:** All GitHub Actions used in this job must match the required references listed in `docs/ci-action-versions.md`. Canonical CI versions: `actions/checkout@v5` and `actions/setup-go@v6`. If you need to change a version, update the manifest first and then adjust every workflow and task doc that depends on it so the project stays consistent.

**Note on Caching:** `actions/setup-go` enables module caching by default when a `go.sum` file exists at the repository root. Do not disable this cache; it should remain active to reduce build times.

## 2. Acceptance Criteria

- The `ci.yml` file contains a job named `test`.
- The job uses `actions/checkout@v5` to check out the repository code.
- It uses `actions/setup-go@v6` to set up the Go environment.
- `actions/setup-go` caching is enabled so dependencies are keyed from the root `go.sum` file and reused between workflow runs.
- It executes `make test`.
- The job fails fast on test failure (non-zero exit code from `make test` causes job to fail).

## 3. Test Plan

- **Integration Test (Passing Tests):**
  1. Push a commit with passing tests to a test branch.
  2. Open a PR to `main`.
  3. Open the GitHub Actions workflow run for the PR.
  4. Confirm the `test` job appears and shows "completed".
  5. Click into the `test` job and confirm the step that runs `make test` succeeded.
  6. Verify the job log contains actual test output (e.g., "ok github.com/...").
  7. Rerun the workflow (or trigger another push) and inspect the second run's `actions/setup-go` logs to confirm the cache key references the hash of `go.sum` and reports a cache restore ("Cache hit" message or noticeably reduced dependency download time).
- **Integration Test (Failing Tests):**
  1. Repeat the above steps with a deliberately failing test.
  2. Confirm the `test` job fails and the job log contains the failing test output.
