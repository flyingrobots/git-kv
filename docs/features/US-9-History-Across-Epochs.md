# Feature Spec: US-9 — History across epochs

**User Story:** As a developer, I can view the history of a key even if it spans multiple epochs.

**Depends On:** `Parent-Epoch` traversal.

## 1. Acceptance Criteria

- A `git kv history <key>` command exists.
- When run, the command shows the commit log for the specified key.
- If the history of the key extends into a previous epoch, the command automatically detects the `Parent-Epoch` trailer in the first commit of the current epoch.
- The command then continues to trace the key's history in the parent epoch's ledger branch.
- The output should be a continuous, unified log of commits affecting the key, regardless of epoch boundaries.

## 2. Test Plan

- **Setup:** Create a key in epoch A. Create a new epoch B. Update the same key in epoch B.
- **Success Case:** Run `git kv history <key>` and verify that the output contains the commits from *both* epoch A and epoch B in the correct chronological order.
- **No Parent:** Test on a repository with only one epoch and verify the command works correctly.
- **Multiple Hops:** Test on a repository with three or more epochs (A, B, C) and verify history can be traced from C back to A.

## 3. Tasks

- [ ] **CLI: `history` command:** Implement the `git kv history <key>` command.
- [ ] **Client: History Logic:** Implement the core logic to get the commit history for a specific file path in a git branch (`git log -- <path>`).
- [ ] **Client: Epoch Traversal:** Enhance the history logic to detect the beginning of an epoch (a commit with no parent or a special epoch-start commit).
- [ ] **Client: `Parent-Epoch` Parsing:** If the beginning is reached, inspect the commit for a `Parent-Epoch` trailer.
- [ ] **Client: Recursive History:** If a parent epoch is found, recursively call the history logic on the parent epoch's ledger branch and prepend the results.
- [ ] **Testing:** Add an integration test for the multi-epoch history traversal scenario.
- [ ] **Documentation:** Document the `history` command and its ability to cross epoch boundaries.
