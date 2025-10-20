# Task: Implement CI Lint Job

**Feature:** `Foundational CI/CD Pipeline`

## 1. Description

Add a job to the CI workflow that checks out the code, sets up the Go environment, and runs the project's linter.

**Note on Action Versions:** All GitHub Actions in this job must match the canonical references recorded in `docs/ci-action-versions.md` (currently `actions/checkout@v5` and `actions/setup-go@v6`). Update the manifest first if these versions ever change so every CI task stays aligned.

## 2. Acceptance Criteria

- The `ci.yml` file contains a job named `lint`.
- The job uses `actions/checkout@v5` to check out the repository code.
- It uses `actions/setup-go@v6` to set up the Go environment.
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
