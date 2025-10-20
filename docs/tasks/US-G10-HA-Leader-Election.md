# Task: HA Leader Election

**Feature:** `US-G10 — HA Failover`

## 1. Description

Implement the leader election mechanism for Stargate. This component ensures that in a multi-node HA setup, only one Stargate instance is active and processing write requests at any given time.

## 2. Acceptance Criteria

- A leader election module is integrated into the Stargate service.
- **Lock Mechanism:** Use Redis `SET` with `NX` (set if not exist) and `EX` (expire) on key `"stargate:leader"` with a 30-second TTL.
- On startup, each Stargate instance attempts to acquire this lock. The instance that successfully acquires the lock becomes the leader.
- **Renewal:** The leader must renew its lock every 10 seconds.
- **Renewal Failure:** On transient renewal failure, retry immediately up to 3 times with 500ms, 1s, 2s exponential backoff. If renewal still fails, the instance must immediately step down, delete its lock (if still owner), and enter a re-acquire loop.
- **Re-acquire Loop:** In the re-acquire loop, attempt to obtain the lock every 5 seconds with exponential backoff up to 60 seconds.
- If the leader fails, its lock expires, allowing a standby instance to acquire the lock and become the new leader.

## 3. Test Plan

- **Integration Test (Failover):**
  1. Start Stargate instance A.
  2. Wait 10s. Assert `/healthz` and `/readyz` report A as leader.
  3. Kill instance A.
  4. Wait 5s for lease timeout.
  5. Start Stargate instance B.
  6. Wait 15s. Assert `/healthz` and `/readyz` report B as leader.
  7. Inspect logs of both A and B to assert there was never an overlap where both instances logged "became leader" within the same lease window.
- **Integration Test (Split-Brain Prevention):**
  1. Deploy two Stargate instances (A and B).
  2. Partition the network between A and the Redis datastore for 10 seconds.
  3. Simultaneously force A to attempt to renew its lock and B to attempt to acquire the lock.
  4. Poll `/readyz` endpoints every 1 second.
  5. Assert that at no point do both A and B report themselves as leader.
  6. Scan logs for leadership grant/revoke events.
- **Unit Test (Lock Acquisition):**
  - Test lock acquisition success/failure (on failure, the instance must not transition to leader).
  - Test concurrent acquisition attempts where only one caller receives the lock.
- **Unit Test (Renewal & Demotion):**
  - Test renewal success.
  - Test renewal failure triggers demotion and cleanup (e.g., deleting its lock if still owner).
