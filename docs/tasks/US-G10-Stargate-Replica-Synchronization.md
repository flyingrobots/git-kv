# Task: Stargate Replica Synchronization

**Feature:** `US-G10 — HA Failover`

## 1. Description

Implement the mechanism for passive Stargate nodes to continuously synchronize their Git repositories with the active leader. This ensures that when a failover occurs, the new leader has the most up-to-date state.

## 2. Acceptance Criteria

- Passive Stargate instances execute `git fetch --mirror <leader_remote>` every 30 seconds, coordinated by a per-replica lock (e.g., `flock` or atomic PID file) so a new fetch starts only after the previous completes; overlapping fetches are skipped rather than queued. Each fetch runs under a watchdog timeout (e.g., 25 seconds) after which the process is terminated, lock released, and a retry scheduled via backoff.
- The `leader_remote` is configured to point to the active Stargate instance.
- **Synchronization SLOs:**
  - Soft SLO: Median replication lag must be <= 15 seconds (half the fetch interval).
  - Hard SLO: 99th percentile lag must be <= 2 minutes. Violations trigger automated alerts (PagerDuty) and require on-call to open an incident, investigate within 15 minutes, and document remediation.
- The synchronization process implements retries with exponential backoff: default 5 attempts, multiplier 2, maximum delay 60 seconds, full jitter. Retry only transient errors (network timeouts, 5xx, transport errors); fail fast on 4xx/authentication failures. Settings are configurable (retry count, multiplier, maxDelay, retryableErrorList).
- The passive node's repository is an eventually-consistent replica with bounded staleness (lag ≤ 2 minutes) rather than a strict mirror.

## 3. Test Plan

- **Integration Test (Bulk Writes):** Deploy leader/passive. Apply 100 sequential `git kv set` operations (unique keys). Poll passive every 1s for 30s and assert parity is achieved within 30s (otherwise fail).
- **Integration Test (Lag Metrics):** Record lag metrics during steady state; verify median ≤15s and P99 ≤120s; confirm alert triggers when thresholds exceeded.
- **Integration Test (Failover):** After convergence, promote passive to leader (documented procedure), switch client traffic, and verify read/write consistency before/during/after promotion with no lost commits or out-of-order histories.
- **Integration Test (Network Disruption):** Use firewall rules or `tc netem` to drop traffic for 10s, 60s, and 5m; confirm sync resumes, lag returns within SLO, and no duplicate commits.
- **Integration Test (Leader Failure Mid-Fetch):** Kill leader during active fetch; ensure passive handles errors, retries per policy, and resyncs once leader returns.
- **Integration Test (Concurrency/Locking):** Simultaneously trigger multiple sync loops; assert only one fetch runs at a time and others skip.
- **Integration Test (Load):** Populate 100k refs and measure sync duration/CPU; ensure process completes within configured timeout.
- **Integration Test (Logs & Metrics):** During disruptions collect logs/metrics and verify they record retries, backoff, and recovery steps.
