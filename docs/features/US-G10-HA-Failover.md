# Feature Spec: US-G10 — HA Failover

**User Story:** As an operator, I can run Stargate in a High Availability configuration to prevent a single point of failure.

**Depends On:** Leader lock, health endpoints, replica sync.

## 1. Acceptance Criteria

- Stargate can be deployed in an Active-Passive model.
- A leader election mechanism (e.g., a file-based lock like `flock` on a shared volume) is used to determine the active node.
- The active node holds the leader lock and serves all write traffic.
- Passive (standby) nodes continuously try to acquire the lock.
- Pushes to a passive node are rejected with a specific exit code (`9` as per spec) and a "not leader" message.
- The passive node continuously mirrors the active node's Git repository (`git fetch --mirror`).
- A floating IP or DNS entry (`stargate.local`) points to the active node.
- Health endpoints (`/healthz`, `/readyz`) are available.
  - `/healthz`: indicates the service is running.
  - `/readyz`: indicates the node is the leader and ready to accept traffic.

## 2. Test Plan

- **Setup:** Deploy two Stargate nodes (one active, one passive) with a shared volume for the leader lock.
- **Failover Test:**
  1. Direct traffic to the active node and perform a successful `git kv set` push.
  2. Stop the active node service.
  3. Verify the passive node acquires the leader lock and becomes active.
  4. Point the floating IP/DNS to the new active node.
  5. Perform another `git kv set` push and verify it succeeds.
- **Passive Rejection:** While the system is stable, try to push directly to the IP of the passive node and verify it is rejected with the correct exit code.

## 3. Tasks

- [ ] **HA Strategy:** Finalize the choice of leader election mechanism (e.g., `flock` on NFS).
- [ ] **Stargate: Leader Election:** Implement the leader election logic in the Stargate service startup sequence.
- [ ] **Stargate: Passive Rejection:** The `pre-receive` hook must check for leadership and exit with code `9` if the node is passive.
- [ ] **Stargate: Replica Sync:** Implement the logic for the passive node to continuously fetch from the active node.
- [ ] **Stargate: Health Endpoints:** Implement the `/healthz` and `/readyz` HTTP endpoints.
- [ ] **Testing:** Create a Docker Compose or similar environment to simulate the two-node HA setup for integration testing.
- [ ] **Testing:** Write an automated script to perform the failover test plan.
- [ ] **Documentation:** Write a detailed runbook for SREs on how to set up, monitor, and manage the HA configuration.
