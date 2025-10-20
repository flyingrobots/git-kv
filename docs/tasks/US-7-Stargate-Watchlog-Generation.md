# Task: Stargate Watchlog Generation

**Feature:** `US-7 — Watch Feed`

## 1. Description

Update the Stargate's `post-receive` hook to generate a commit in the `refs/kv-watchlog/<ns>@<epoch>` ref for every accepted ledger change. This creates an append-only feed of changes that clients can subscribe to.

## 2. Acceptance Criteria

- The `post-receive` hook logic is triggered after a successful push to a ledger ref.
- The hook creates a new commit.
- The commit's content should be a blob containing structured data (e.g., JSON) about the change, including the new ledger commit OID and a timestamp.
- This new commit is made on the appropriate `refs/kv-watchlog/...` branch.
- The watchlog has a linear history; each new commit has the previous watchlog commit as its parent.
- The Stargate's mirror configuration is updated to push `refs/kv-watchlog/*` to the GitHub remote.

## 3. Test Plan

- **Integration Test:** Perform a `git kv set` operation. After the push succeeds, inspect the test Stargate repo and verify that a new commit has been added to the watchlog ref.
- **Integration Test:** Verify the content of the new watchlog commit blob is correctly formatted JSON containing the correct ledger OID.
- **Integration Test:** Check the mock GitHub remote and verify the watchlog ref was mirrored correctly.
