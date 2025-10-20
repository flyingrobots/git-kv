# Task: Sync Logic: Force and Admin Check

**Feature:** `US-G11 — Split-Brain Recovery`

## 1. Description

Implement the `--force` mode for `git kv stargate sync`, which allows an authorized administrator to forcibly overwrite the GitHub mirror's state to match the Stargate. This is a powerful and potentially destructive operation, requiring strict authorization.

## 2. Acceptance Criteria

- **`--force` mode:**
  - When invoked, it performs a `git push --force <mirror_remote> <refspec>` for all diverging refs where the Stargate is the source of truth.
  - This operation will overwrite any commits on the mirror that are not present on the Stargate.
- **Admin Check:**
  - Before executing `--force`, the command verifies that the user running it is listed in the `stargate.admins` section of the `.kv/policy.yaml` file.
  - If the user is not an authorized admin, the command is rejected with an error.

## 3. Test Plan

- **Integration Test (Force Success):** Simulate a split-brain scenario where the mirror has diverged. Run `sync --force` as an authorized admin and verify the mirror is forcibly updated to match the Stargate.
- **Integration Test (Admin Failure):** Simulate the same split-brain scenario. Run `sync --force` as a non-authorized user and verify the command is rejected with an authorization error.
