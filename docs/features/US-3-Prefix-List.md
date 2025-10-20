# Feature Spec: US-3 — Prefix list (index-only)

**User Story:** As a developer, I can list keys by prefix quickly, without the client needing to scan the entire repository.

**Depends On:** `I` (Index) writer/reader.

## 1. Acceptance Criteria

- A `git kv list --prefix <p>` command exists.
- The command **does not** perform a full checkout or scan of the `refs/kv/<ns>` (Ledger) branch.
- Instead, it reads entries from the `refs/kv-index/<ns>` (Index) branch.
- The lookup should be efficient, using the prefix-sharded directory structure of the index (e.g., `index/<pp>/<pp>/...`).
- The command outputs a list of key names matching the prefix, one per line.
- Keys that have been deleted (i.e., have a tombstone `.ref` pointer in the index) are not included in the output.
- Keys that have expired due to TTL (if TTL is implemented) are not included in the output.

## 2. Test Plan

- **Success Case:** Set several keys with a common prefix (e.g., `user:1:`, `user:2:`) and some without. Run `git kv list --prefix user:` and verify only the correct keys are returned.
- **No Match:** Run the command with a prefix that matches no keys and verify it returns an empty list.
- **Tombstone:** Set a key, then delete it. Run `list` with the key's prefix and verify it does not appear in the list.
- **Performance:** Create 1 million keys, with 1,000 under a specific prefix. Measure the time for `git kv list --prefix ...` and verify it is significantly faster than any operation that would require a full repo scan.

## 3. Tasks

- [ ] **CLI: `list` command:** Implement the `git kv list` command structure, including the `--prefix` flag.
- [ ] **Client: Index Reader:** Implement the logic to read the `refs/kv-index/<ns>` branch.
- [ ] **Client: Prefix Traversal:** Implement the logic to efficiently traverse the index's directory structure based on the given prefix.
- [ ] **Client: Pointer Parsing:** Implement logic to read the `.ref` pointer files and filter out any that are marked as `deleted`.
- [ ] **Testing:** Add unit tests for the prefix traversal and pointer parsing logic.
- [ ] **Testing:** Add an integration test that creates a set of keys and tombstones and verifies the `list` command returns the correct subset.
- [ ] **Documentation:** Document the `git kv list` command in the `README.md`.
