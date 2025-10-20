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

- **Integration Test (Success):** Push a commit with passing tests. Verify the `test` job in the CI pipeline passes.
- **Integration Test (Failure):** Push a commit with a deliberately failing test. Verify the `test` job in the CI pipeline fails.
