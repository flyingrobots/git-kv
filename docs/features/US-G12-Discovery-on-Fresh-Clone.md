# Feature Spec: US-G12 — Discovery on Fresh Clone

**User Story:** As a developer on a fresh clone, I can discover the Stargate push URL automatically.

**Depends On:** `git kv remote setup`, `policy.stargate.push_url`.

**Note:** This User Story is largely a subset of `US-G1 — Provision Stargate + Bootstrap`. This document focuses on the client-side discovery aspect.

## 1. Acceptance Criteria

- A developer clones the repository from GitHub for the first time.
- The repository contains a `.kv/policy.yaml` file with a `stargate.push_url` defined.
- The developer runs `git kv remote setup`.
- The command finds and reads the `.kv/policy.yaml` file from the local clone.
- It extracts the `stargate.push_url` value.
- It configures the git remote by running `git config remote.origin.pushurl <url>`.
- The developer receives a confirmation message.

## 2. Test Plan

- **Success Case:** Create a fresh clone of a repo containing a policy file. Run `git kv remote setup`. Use `git remote -v` to verify that the `push` URL for `origin` has been correctly set, while the `fetch` URL remains the same.
- **Failure Case (No Policy):** Clone a repo, but delete the `.kv/policy.yaml` file. Run `git kv remote setup` and verify it fails with a clear error message.
- **Failure Case (No Key):** Clone a repo, but edit the policy file to remove the `push_url` key. Run `git kv remote setup` and verify it fails with a clear error message.

## 3. Tasks

- [ ] **CLI: `remote setup` command:** Implement the command in the Go CLI.
- [ ] **Client: Policy Locator:** Implement logic to find the `.kv/policy.yaml` file in the root of the repository.
- [ ] **Client: YAML Parser:** Implement logic to parse the YAML file and safely extract the `stargate.push_url` value.
- [ ] **Client: Git Config:** Implement the logic that executes `git config` to update the `pushurl`.
- [ ] **Testing:** Add an integration test that creates a temporary git repository with a policy file, runs the command, and asserts that the git configuration is correct.
- [ ] **Documentation:** Ensure the `README.md` Quick Start section clearly instructs new users to run this command after cloning.
