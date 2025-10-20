# Task: Implement CI Lint Job

**Feature:** `Foundational CI/CD Pipeline`

## 1. Description

Add a job to the CI workflow that checks out the code, sets up the Go environment, and runs the project's linter.

**Note on Action Versions:** All GitHub Actions used (e.g., `actions/checkout`, `actions/setup-go`) should be pinned to specific major versions (e.g., `@v4`, `@v5`) for stability. Consistency in these versions across all CI-related tasks is paramount to avoid maintenance burden and ensure predictable behavior.

## 2. Acceptance Criteria

- The `ci.yml` file contains a job named `lint`.
- The job uses `actions/checkout@v4` to check out the repository code.
- It uses `actions/setup-go@v5` to set up the Go environment.
- It executes `make lint`.

## 3. Test Plan

- **Integration Test (Success):**
  1. Push a commit with no linting errors to a test branch.
  2. Open a PR to `main`.
  3. Open the GitHub Actions workflow run for the PR.
  4. Confirm the `lint` job appears and shows "completed".
  5. Click into the `lint` job and confirm the step that runs `make lint` succeeded.
  6. Verify the job log contains lint output (e.g., "No issues found" or similar).
- **Integration Test (Failure):**
  1. Repeat the above steps with a deliberately introduced linting error.
  2. Confirm the `lint` job fails and the job log contains the lint error output.
