# Task: Add CI Status Badge to README

**Feature:** `Foundational CI/CD Pipeline`

## 1. Description

Add a GitHub Actions status badge to the `README.md` file to visually indicate the current build status of the `main` branch.

## 2. Acceptance Criteria

- The `README.md` file is updated.
- It includes a Markdown snippet for a GitHub Actions status badge: `[![CI Status](https://github.com/flyingrobots/git-kv/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/flyingrobots/git-kv/actions/workflows/ci.yml)`.
  ```markdown
  [![CI Status](https://github.com/flyingrobots/git-kv/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/flyingrobots/git-kv/actions/workflows/ci.yml)
  ```
- The badge correctly links to the CI workflow and displays its current status.
- Place the badge prominently near the top of `README.md` so the current status is immediately visible and the link to the workflow is obvious.

## 3. Test Plan

- **Manual Review:** After the PR is merged and the CI pipeline runs, verify that the badge appears correctly on the GitHub repository's `README.md` page.
- **Manual Review:** Verify that the badge accurately reflects the status of the latest build on the `main` branch.
