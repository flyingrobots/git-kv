# Feature Spec: US-7 — Watch Feed

**User Story:** As a developer or service, I can watch a feed of changes to react to updates.

**Depends On:** Watchlog per epoch written in post-receive; mirrored.

## 1. Acceptance Criteria

- The Stargate's `post-receive` hook writes a record to a `refs/kv-watchlog/<ns>@<epoch>` ref for every change.
- This ref is an append-only log, where each commit adds a new entry.
- The entry contains metadata about the change (e.g., the ledger commit OID, timestamp, namespace).
- A `git kv watch [--since <oid>]` command exists.
- The command can stream the contents of the watchlog to the user.
- The `--since` flag allows the user to start streaming only the changes that have occurred after a specific watchlog commit OID.
- The watchlog refs are mirrored to GitHub, allowing clients to watch for changes without needing access to the Stargate.

## 2. Test Plan

- **Success Case:** Make several changes with `git kv set`. Run `git kv watch` and verify all changes are present in the output.
- **`--since` Flag:** Get an OID from the middle of the watchlog. Run `git kv watch --since <OID>` and verify only subsequent changes are shown.
- **Mirroring:** Verify that the `refs/kv-watchlog/...` refs are correctly mirrored to the GitHub remote after changes are made.

## 3. Tasks

- [ ] **Stargate: `post-receive` Hook:** Update the hook to write a new commit to the appropriate `refs/kv-watchlog/...` ref for each change.
- [ ] **Stargate: Mirroring:** Ensure the `refspec` for the mirror remote includes `refs/kv-watchlog/*`.
- [ ] **CLI: `watch` command:** Implement the `git kv watch` command.
- [ ] **Client: Watch Logic:** Implement the logic to read the watchlog ref and stream its contents.
- [ ] **Client: `--since` Logic:** Implement the `--since` flag to allow starting from a specific point in the log.
- [ ] **Testing:** Add an integration test that makes several changes and then uses `watch` to verify the log content.
- [ ] **Documentation:** Document the `watch` command and the concept of the watchlog.
