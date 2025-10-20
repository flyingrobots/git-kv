# Task: Stargate Replica Synchronization

**Feature:** `US-G10 — HA Failover`

## 1. Description

Implement the mechanism for passive Stargate nodes to continuously synchronize their Git repositories with the active leader. This ensures that when a failover occurs, the new leader has the most up-to-date state.

## 2. Acceptance Criteria

- Passive Stargate instances periodically execute `git fetch --mirror <leader_remote>`.
- The `leader_remote` is configured to point to the active Stargate instance.
- The synchronization process is robust and handles network interruptions.
- The passive node's repository is always a mirror of the active node's repository, with minimal lag.

## 3. Test Plan

- **Integration Test:** Deploy two Stargate instances (one leader, one passive). Perform several write operations on the leader. Verify that the passive node's repository eventually reflects all changes from the leader.
- **Integration Test:** Measure the synchronization lag between the leader and the passive node and ensure it meets acceptable SLOs.
- **Edge Case:** Simulate network disconnection between leader and passive. Verify that synchronization resumes correctly once the connection is restored.
