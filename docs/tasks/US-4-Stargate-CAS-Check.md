# Task: Stargate-CAS-Check in `pre-receive`

**Feature:** `US-4 — CAS on branch`

## 1. Description

Update the Stargate's `pre-receive` hook to perform the Compare-And-Swap (CAS) check. When a client provides the `--expect-tree` value, the hook must verify that the current state of the branch matches the expected state before allowing the update.

## 2. Acceptance Criteria

- The client **must** send the `--expect-tree` OID via a Git push option. The exact format to use is `-o expect-tree=<oid>`.
- Before other validation, the hook checks if a CAS operation is requested.
- If so, it reads the current OID of the `refs/kv/<ns>` branch tip.
- It compares this current OID with the expected OID from the client.
- If they do not match, the hook rejects the push with a specific, machine-readable error message: "ERROR: CAS_CONFLICT: Expected tree OID mismatch." This rejection must use exit code 2.
- If they do match, validation continues as normal.

## 3. Test Plan

All integration tests run against a disposable bare repository hosted by Stargate or a local mock that faithfully enforces the `pre-receive` hook. Use namespace `refs/kv/test-ns`. After each scenario, reset the repo state (`git push origin --delete refs/kv/test-ns` if created, recreate baseline branch) to keep tests isolated.

### Unit Tests (Hook Logic)
- Inject malformed `expect-tree` options (missing value, uppercase hex, too short/long). Assert the hook rejects the push immediately, returns exit code `2`, and emits `ERROR: CAS_CONFLICT: Expected tree OID mismatch.` without touching refs and before any downstream validation runs.
- Mock absence of state: when the incoming push targets a nonexistent `refs/kv/<ns>` tip and `expect-tree` equals `000...0`, ensure the hook treats it as matching and allows subsequent validation to run.
- Mock mismatch scenario: hook sees current OID `abc...123`, `expect-tree` `def...456`; ensure rejection with exit `2` and error message.
- Ensure the hook runs before downstream validators by injecting a fake validator that records whether CAS was checked first; assert CAS check occurs prior to other validations.

### Integration Tests (Acceptance)
1. **Happy Path Existing Ref:**
   - `git push origin HEAD:refs/kv/test-ns -o expect-tree=<current tip>`.
   - Expect exit `0` and ref updated.
2. **Happy Path Initial Creation:**
   - Delete `refs/kv/test-ns`; push new commit with `-o expect-tree=0000000000000000000000000000000000000000`.
   - Expect success and ref created.
3. **Malformed OID:**
   - `git push origin HEAD:refs/kv/test-ns -o expect-tree=deadbeef`.
   - Expect exit `2`, error message, and branch unchanged.
4. **Race Condition Loss:**
   - Client A records current OID `A`.
   - Client B pushes new commit moving tip to `B`.
   - Client A pushes with `-o expect-tree=A`; expect rejection with CAS error and tip remains `B`.
5. **Rapid Repeated Races:**
   - Loop 10 times: record tip, push conflicting commit from another process, attempt push with stale OID via `-o expect-tree`. Ensure each attempt fails with CAS error and no unexpected success occurs; verify final tip equals last successful commit.
6. **Concurrent Different Namespaces:**
   - Simultaneously push to `refs/kv/test-ns` and `refs/kv/other-ns` with valid expect-tree values. Ensure independent CAS checks do not interfere and both succeed when expected.
7. **Validation Ordering:**
- Configure a downstream validator to always fail after CAS succeeds. Push with correct `expect-tree`; expect CAS passes, validator failure rejects the push with validator’s message (e.g., `ERROR: POLICY_BLOCKED`), and refs remain unchanged.

### Integration Tests (Rollback & Consistency)
- After each rejection, verify the tip OID equals the pre-test value by running `git ls-remote origin refs/kv/test-ns`.
- Confirm no reflog entries or temporary refs are left behind.
- For concurrency tests, run `git fsck` to ensure repo integrity.

### Automation Notes
- Mark unit tests to run in CI on every push (fast suite).
- Integration race tests may be slower; tag them as `race` and run nightly or on demand (`make test-cas-race`). Document commands in test harness.
