# Feature Spec: US-8 — Policy Enforcement

**User Story:** As an operator, I can enforce policies on keys, signers, and values.

**Depends On:** `H.pre-receive`; signer checks.

## 1. Acceptance Criteria

- The Stargate server loads a `.kv/policy.yaml` file.
- The `pre-receive` hook enforces policies defined in this file for every push.
- **Signer Verification:** If `require_signed_commits` is true, the hook verifies that the commit is signed by a key listed in the `writers` section.
- **Prefix Enforcement:** The hook checks every key in a transaction against the `allowed_prefixes` list and rejects the push if a key does not have a permitted prefix.
- **Size Limits:** The hook checks the size of values against `max_value_inline` and `max_value_total` and rejects if they exceed the limits.
- **LFS Prevention:** The hook inspects the push content and rejects it if it contains Git LFS pointers, if `lfs: true` is in the `forbidden` section.
- Rejections due to policy violations provide clear, specific error messages to the user.

## 2. Test Plan

- **Signed Commits:** Enable `require_signed_commits`. Try to push an unsigned commit and verify it's rejected. Push a signed commit from an allowed key and verify it's accepted.
- **Prefixes:** Define a strict `allowed_prefixes` list. Try to `set` a key with a forbidden prefix and verify the push is rejected.
- **Size Limits:** Set a low `max_value_total`. Try to push a value larger than the limit and verify it's rejected.
- **LFS:** Try to push a commit containing a Git LFS pointer and verify it's rejected.

## 3. Tasks

- [ ] **Stargate: Policy Loader:** Implement logic for the Stargate service to load and parse the `.kv/policy.yaml` file.
- [ ] **Stargate: Pre-receive Hook Update:** Integrate the policy checks into the `pre-receive` hook logic.
- [ ] **Stargate: Signer Check:** Implement `git verify-commit` logic and check the signer's key against the policy's `writers` list.
- [ ] **Stargate: Prefix Check:** Implement logic to inspect the transaction's keys and validate them against `allowed_prefixes`.
- [ ] **Stargate: Size Check:** Implement logic to check value sizes against the policy limits.
- [ ] **Stargate: LFS Check:** Implement logic to detect and reject LFS pointers.
- [ ] **Testing:** Add integration tests for each policy violation scenario (bad signature, bad prefix, oversized value, LFS pointer).
- [ ] **Documentation:** Fully document all available policy options in the `SPEC.md` or a dedicated `POLICY.md` file.
