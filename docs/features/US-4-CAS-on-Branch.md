# Feature Spec: US-4 — CAS on branch

**User Story:** As a developer, I can perform a Compare-And-Swap operation at the branch level to prevent lost updates.

**Depends On:** Linear ledger; `H.FF enforcement`.

## 1. Acceptance Criteria

- The `git kv set` and `git kv mset` commands accept an `--expect-tree <OID>` argument.
- When this flag is provided, the client generates a commit as usual.
- The client then uses a Git push option or similar mechanism to inform the Stargate of the expected tree OID.
- The Stargate's `pre-receive` hook checks if the current `HEAD` of the `refs/kv/<ns>` branch matches the provided `<OID>`.
- If it matches, the push is accepted (assuming all other validation passes).
- If it does not match, the push is rejected with a clear error message indicating a CAS conflict (a race condition where another commit landed first).
- The CLI should translate this specific rejection into a user-friendly error and exit with a distinct exit code (e.g., `2` as per the spec).

## 2. Test Plan

- **Success Case:** Get the current tree OID of the ledger branch. Run `git kv set --expect-tree <OID> ...`. Verify the push succeeds.
- **Failure Case:** Get the current tree OID. In one terminal, run `git kv set` to create a new commit. In a second terminal, try to run `git kv set --expect-tree <original_OID> ...`. Verify the second push is rejected with a CAS conflict error.

## 3. Tasks

- [ ] **CLI: `--expect-tree` flag:** Add the `--expect-tree` flag to the `set` and `mset` commands.
- [ ] **Client: Push Options:** Determine the best mechanism to pass the expected OID to the Stargate. Git's `push --atomic --push-option` is a possibility, but might not be suitable for this. An alternative is to add it to the commit message trailers.
- [ ] **Stargate: Pre-receive Hook Update:** Modify the `pre-receive` hook to read the expected OID.
- [ ] **Stargate: CAS Check:** Implement the logic in the hook to compare the expected OID with the current branch tip and reject the push on mismatch.
- [ ] **CLI: Error Handling:** Implement client-side logic to detect the specific CAS rejection from Stargate and return the correct exit code and error message.
- [ ] **Testing:** Add an integration test that simulates the CAS failure scenario described in the test plan.
- [ ] **Documentation:** Document the `--expect-tree` flag and its use for preventing race conditions.
