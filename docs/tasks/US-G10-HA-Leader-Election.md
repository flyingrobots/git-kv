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

- **Unit Test:** Test the lock acquisition and release logic in isolation.
- **Integration Test (Failover):** Deploy two Stargate instances. Verify that one becomes the leader. Simulate a leader crash and verify that the other instance takes over leadership.
- **Integration Test (Split-Brain Prevention):** Verify that under no circumstances do two instances simultaneously believe they are the leader.
