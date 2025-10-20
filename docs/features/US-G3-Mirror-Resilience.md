# Feature Spec: US-G3 — Mirror Resilience & Read-After-Write

**User Story:** As a dev/CI, I can fence on GitHub visibility or read from Stargate.

**Depends On:** `kv-mirror/<ns>` watermark; `git kv wait`; optional `--read-from=stargate`.

## 1. Acceptance Criteria

- **Stargate Mirroring:**
  - The Stargate's `post-receive` hook queues a mirroring job for every successful push.
  - A background worker on the Stargate pushes the updated refs to the GitHub mirror.
  - Upon successful push to the mirror, the worker updates a `refs/kv-mirror/<ns>` ref on GitHub to point to the OID of the just-pushed ledger commit.
- **`git kv wait` command:**
  - A `git kv wait --oid <hash> --visible-on=github` command exists.
  - The command polls the GitHub remote (e.g., via `git ls-remote`) for the `refs/kv-mirror/<ns>` ref.
  - It exits with success (0) only when the mirror ref's OID is equal to or an ancestor of the specified `<hash>`.
  - The command has a configurable timeout (`--timeout`) and exits with an error if the timeout is reached.
- **`git kv get --read-from=stargate`:**
  - The `get` command accepts a `--read-from=stargate` flag.
  - When specified, the client fetches the key directly from the Stargate's Git repo instead of the default `origin` (GitHub).
  - This requires the client to have read access to the Stargate repo.

## 2. Test Plan

- **Stargate Mirroring:** Test that a successful push to Stargate is mirrored to GitHub and the `kv-mirror` ref is updated correctly.
- **`wait` Success:** Run `git kv wait` after a push and verify it exits successfully once the mirror ref is updated.
- **`wait` Timeout:** Run `git kv wait` for an OID that will never appear and verify the command times out and exits with a non-zero code.
- **`get` from Stargate:** Configure a key, push it, and immediately try to `get` it using `--read-from=stargate`. Verify it returns the correct value while a normal `get` might (transiently) fail or see an older value.

## 3. Tasks

- [ ] **Stargate: `post-receive` Hook:** Implement the hook to queue mirroring jobs (e.g., write to a journal file).
- [ ] **Stargate: Mirror Worker:** Implement the background worker process that reads the journal and pushes to the GitHub remote.
- [ ] **Stargate: Watermark:** The mirror worker must update the `refs/kv-mirror/<ns>` ref on the GitHub remote after a successful push.
- [ ] **CLI: `wait` command:** Implement the `git kv wait` command, including the polling logic using `git ls-remote`.
- [ ] **CLI: `get --read-from`:** Add the `--read-from=stargate` flag and the associated logic to the `get` command to fetch from the `pushurl` remote instead of the `url` remote.
- [ ] **Testing:** Add an integration test for the end-to-end mirror-and-wait flow.
- [ ] **Testing:** Add a unit test for the `get --read-from=stargate` logic.
- [ ] **Documentation:** Document the RYW options (`wait` and `read-from`) in the `README.md`.
