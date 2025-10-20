# Task: Implement CI Test Job

**Feature:** `Foundational CI/CD Pipeline`

## 1. Description

Add a job to the CI workflow that checks out the code, sets up the Go environment, and runs the project's tests.

## 2. Acceptance Criteria

- The `ci.yml` file contains a job named `test`.
- The job uses `actions/checkout@v4` to check out the repository code.
- It uses `actions/setup-go@v5` to set up the Go environment.
- It executes `make test`.
- The job reports its status (success/failure) back to GitHub.

## 3. Test Plan

- **Integration Test (Passing Tests):**
  1. Push a commit with passing tests to a test branch.
  2. Open a PR to `main`.
  3. Open the GitHub Actions workflow run for the PR.
  4. Confirm the `test` job appears and shows "completed".
  5. Click into the `test` job and confirm the step that runs `make test` succeeded.
  6. Verify the job log contains actual test output (e.g., "ok github.com/...").
- **Integration Test (Failing Tests):**
  1. Repeat the above steps with a deliberately failing test.
  2. Confirm the `test` job fails and the job log contains the failing test output.
