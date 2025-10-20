# Feature Spec: US-G11 — Split-Brain Recovery

**User Story:** As an operator, I can safely recover from a split-brain scenario where the GitHub mirror has diverged from the Stargate.

**Depends On:** `git kv stargate sync` tooling, admin policy.

## 1. Acceptance Criteria

- An admin command `git kv stargate sync` exists.
- The command can detect differences between the refs on the Stargate (source of truth) and the GitHub mirror.
- `sync --dry-run`: This mode prints the refs that have diverged but takes no action.
- `sync --repair`: This mode attempts a fast-forward push to the mirror for any refs that are behind. It will fail if the mirror has commits that the Stargate does not (a non-fast-forward situation).
- `sync --force`: This mode can only be run by an admin specified in the policy file. It will forcibly overwrite the mirror's refs to match the Stargate's, even if it means losing commits on the mirror. This is the tool to fix split-brain.
- The command provides clear and detailed output about the actions it is taking.

## 2. Test Plan

- **Setup:** Manually push a commit directly to the GitHub mirror to create a divergence (a split-brain scenario).
- **Dry Run:** Run `sync --dry-run` and verify it correctly reports the diverged ref.
- **Repair Failure:** Run `sync --repair` and verify that it fails because the update is not a fast-forward.
- **Force Success:** Run `sync --force` as a designated admin and verify the mirror ref is now identical to the Stargate ref.
- **Admin Check:** Try to run `sync --force` as a non-admin user and verify the command is rejected.

## 3. Tasks

- [ ] **CLI: `stargate sync` command:** Implement the `git kv stargate sync` command with its sub-modes (`--dry-run`, `--repair`, `--force`).
- [ ] **Sync Logic: Diff:** Implement logic to compare the refs between the Stargate and the mirror remote.
- [ ] **Sync Logic: Repair:** Implement the fast-forward push logic for `--repair` mode.
- [ ] **Sync Logic: Force:** Implement the force-push logic for `--force` mode.
- [ ] **Sync Logic: Admin Check:** Implement the check to ensure the user running `--force` is listed in the `stargate.admins` section of the policy file.
- [ ] **Testing:** Add an integration test that simulates a split-brain scenario and tests all modes of the `sync` command.
- [ ] **Documentation:** Write a clear warning in the documentation about the danger of `--force` and the importance of the `stargate.admins` policy.
