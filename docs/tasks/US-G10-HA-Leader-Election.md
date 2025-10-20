# Task: HA Leader Election

**Feature:** `US-G10 — HA Failover`

## 1. Description

Implement the leader election mechanism for Stargate. This component ensures that in a multi-node HA setup, only one Stargate instance is active and processing write requests at any given time.

## 2. Acceptance Criteria

- A leader election module is integrated into the Stargate service.
- On startup, each Stargate instance attempts to acquire a distributed lock (e.g., using `flock` on a shared file system, or a dedicated consensus service).
- The instance that successfully acquires the lock becomes the leader.
- The leader periodically renews its lock to maintain leadership.
- If the leader fails, its lock is released (or expires), allowing a standby instance to acquire the lock and become the new leader.

## 3. Test Plan

- **Unit Test:** Test the lock acquisition and release logic in isolation.
- **Integration Test (Failover):** Deploy two Stargate instances. Verify that one becomes the leader. Simulate a leader crash and verify that the other instance takes over leadership.
- **Integration Test (Split-Brain Prevention):** Verify that under no circumstances do two instances simultaneously believe they are the leader.
