# Feature Spec: US-G2 — Atomic transaction via Stargate

**User Story:** As a dev, my `mset` lands atomically with fast feedback.

**Depends On:** Attestation trailers; `H.pre-receive`.

## 1. Acceptance Criteria

- A `git kv mset --file <batch.jsonl>` command exists.
- The command reads a JSONL file where each line is a `set` or `del` operation.
- The client constructs a new ledger tree and index tree based on the operations.
- The client generates a commit with the appropriate `KV-Txn`, `KV-Index-Tree`, and other attestation trailers.
- The client performs an atomic push of the new ledger and index refs to the Stargate.
- The Stargate's `pre-receive` hook validates the attestation trailers in O(1) time.
- If validation passes, the push is accepted and the developer receives immediate success feedback.
- If validation fails, the push is rejected, and the developer receives a clear error message explaining the mismatch.
- The entire set of operations in the batch file must succeed or fail as a single atomic unit.

## 2. Test Plan

- **Success Case:** Create a batch file with multiple `set` and `del` operations. Run `git kv mset` and verify that the push succeeds and all keys are updated as expected.
- **Failure Case (Attestation Mismatch):** Manually craft a commit with an incorrect `KV-Index-Tree` trailer and push it. Verify the Stargate `pre-receive` hook rejects it.
- **Failure Case (Malformed Input):** Provide a non-existent file or a file with invalid JSONL to `git kv mset` and verify the client CLI exits with a helpful error.
- **Edge Case (Empty File):** Run the command with an empty batch file. The command should succeed and result in no new commit.
- **Edge Case (Large Batch):** Test with a batch file containing thousands of operations to ensure performance.

## 3. Tasks

- [ ] **CLI:** Implement the `git kv mset` command structure in the Go CLI.
- [ ] **Client-Side Tree Construction:** Implement logic for the client to read a batch file and construct the new ledger and index Git trees in memory.
- [ ] **Client-Side Commit Generation:** Implement logic to generate the ledger commit with all required attestation trailers (`KV-Txn`, `KV-Index-Tree`, etc.).
- [ ] **Client-Side Push:** Implement the atomic multi-ref push logic (`git push origin <ledger_ref> <index_ref>`).
- [ ] **Stargate: Pre-receive Hook:** Create the `pre-receive` hook script on the Stargate server.
- [ ] **Stargate: Attestation Validation:** Implement the core O(1) validation logic within the hook that compares commit trailers to the actual push content.
- [ ] **Testing:** Add unit tests for client-side tree construction and commit generation.
- [ ] **Testing:** Add an integration test that sets up a mock Stargate and tests a full `mset` transaction, including success and failure cases.
- [ ] **Documentation:** Update `README.md` and `SPEC.md` with details on the `mset` command usage.
