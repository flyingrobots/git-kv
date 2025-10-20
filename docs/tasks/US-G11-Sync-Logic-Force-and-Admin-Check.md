# Task: Sync Logic: Force and Admin Check

**Feature:** `US-G11 — Split-Brain Recovery`

## 1. Description

Implement the `--force` mode for `git kv stargate sync`, which allows an authorized administrator to forcibly overwrite the GitHub mirror's state to match the Stargate. This is a powerful and potentially destructive operation, requiring strict authorization.

## 2. Acceptance Criteria

- **`--force` mode:**
  - When invoked, it performs a `git push --force <mirror_remote> <refspec>` for all diverging refs where the Stargate is the source of truth.
  - **Diverging Definition:** A ref is "diverging" if the Stargate and mirror OIDs differ and neither commit is an ancestor of the other—there is no fast-forward path in either direction.
  - **Mirror-Only Refs:** Refs present only on the mirror (not on Stargate) are deleted from the mirror when `--delete-mirror-only` is true. The default is **false** (safe mode); enabling deletion requires `--delete-mirror-only=true --confirm-delete-mirror-refs`. Usage examples:
    - Safe (default): `git kv stargate sync --force` → mirror-only refs retained.
    - Destructive (opt-in): `git kv stargate sync --force --delete-mirror-only=true --confirm-delete-mirror-refs` → mirror-only refs deleted.
  - **Read-Only/Protected Refs:** Controlled by `--fail-on-protected-ref=<mode>` (default `abort`):
    - `abort`: abort on first protected ref, exit non-zero (e.g., `... --fail-on-protected-ref=abort`).
    - `skip`: skip protected refs, continue, exit 0 with warnings (e.g., `... --fail-on-protected-ref=skip`).
    - `warn`: skip protected refs, continue, exit 0; warnings logged (e.g., `... --fail-on-protected-ref=warn`).
    - `error-on-any`: skip protected refs but exit non-zero with detailed report (e.g., `... --fail-on-protected-ref=error-on-any`).
    - Modes `abort` and `error-on-any` return non-zero exit codes; `skip` and `warn` return 0 while surfacing warnings.
  - This operation will overwrite any commits on the mirror that are not present on the Stargate.
- **Admin Check:**
  - Before executing `--force`, the command verifies that the user running it is listed in the `stargate.admins` section of the `.kv/policy.yaml` file.
  - If the user is not an authorized admin, the command is rejected with an error.

## 3. Test Plan

- **Integration Test (Force Success):** Diverged refs; run `sync --force` as admin with default `abort` mode; verify mirror matches Stargate and exit 0.
- **Integration Test (Admin Failure):** Run `sync --force` as unauthorized user or with missing `.kv/policy.yaml`; expect authorization error and exit non-zero.
- **Integration Test (Mirror-Only Deletion Opt-In):** Create mirror-only refs; run `sync --force --delete-mirror-only=true --confirm-delete-mirror-refs`; ensure deletions occur; verify absence when confirmation missing.
- **Integration Test (Mirror-Only Safe Default):** With mirror-only refs present, run `sync --force` using defaults; assert no deletions occur and report highlights retained refs.
- **Integration Test (Protected Refs Abort):** Mark mirror ref protected; run with default `abort`; assert operation stops, exit non-zero, no other refs processed.
- **Integration Test (Protected Refs Skip/Warn/Error):** Repeat with modes `skip`, `warn`, `error-on-any`; validate outputs and exit codes (0 for skip/warn, non-zero for error-on-any) and confirm logs include warnings/errors as appropriate.
- **Integration Test (Non-Diverging Refs):** Refs already fast-forwardable; ensure `--force` does not push and logs no overwrites.
- **Integration Test (Mixed Scenarios):** Combination of diverging, protected, mirror-only refs; verify deletions only with confirmation, protected handled per mode, others force-pushed.
- **Unit Test (Flag Parsing):** Validate defaults (`--delete-mirror-only` false, `--fail-on-protected-ref=abort`), confirmation requirements, and invalid mode values.
