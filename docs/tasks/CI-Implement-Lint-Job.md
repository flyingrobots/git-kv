# Task: Implement CI Lint Job

**Feature:** `Foundational CI/CD Pipeline`

## 1. Description

Add a job to the CI workflow that checks out the code, sets up the Go environment, and runs the project's linter.

## 2. Acceptance Criteria

- The `ci.yml` file contains a job named `lint`.
- The job uses `actions/checkout@v4` to check out the repository code.
- It uses `actions/setup-go@v5` to set up the Go environment.
- It executes `make lint`.
- The job reports its status (success/failure) back to GitHub.

## 3. Test Plan

- **Integration Test (Success):** Push a commit with no linting errors. Verify the `lint` job in the CI pipeline passes.
- **Integration Test (Failure):** Push a commit with a deliberate linting error. Verify the `lint` job in the CI pipeline fails.
