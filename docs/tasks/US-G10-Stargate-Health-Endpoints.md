# Task: Stargate Health Endpoints

**Feature:** `US-G10 — HA Failover`

## 1. Description

Implement standard HTTP health endpoints (`/healthz` and `/readyz`) for the Stargate service. These endpoints are crucial for load balancers, orchestrators (like Kubernetes), and monitoring systems to determine the health and readiness of Stargate instances.

## 2. Acceptance Criteria

- The Stargate service exposes an HTTP server (e.g., on a configurable port).
- A `/healthz` endpoint is available.
  - It returns HTTP 200 OK only when all of the following checks succeed:
    1. **Process liveness:** Stargate main loop is running (e.g., self heartbeat timestamp within 5s).
    2. **Git repository access:** `git rev-parse HEAD` (or equivalent libgit call) against the local repo succeeds.
    3. **Filesystem writability:** Required paths (`.stargate/`, journal, checkpoints, logs) are writable (touch + delete test file or equivalent check).
    4. **Configuration readability:** `.kv/policy.yaml` (and other required config files) can be read and parsed successfully.
    5. **Environment variables:** Mandatory env vars (e.g., `STARGATE_HOME`, `GIT_SSH_COMMAND`) are present and non-empty.
    6. **Network connectivity:** TCP connection to dependent services (e.g., mirror remote `github-origin` SSH endpoint, any database host) succeeds within 2s.
    7. **Resource thresholds:** Available disk space on the data volume ≥ 1 GB; available memory ≥ 200 MB (check via system API).
    8. **External services:** Any configured databases or queues respond to a ping/health call with success (auth included).
  - Any failed check returns HTTP 500 with a JSON payload listing failing checks and error messages.
- A `/readyz` endpoint is available.
  - It returns HTTP 200 OK only if **all** of the following are true:
    1. The Stargate instance is the elected leader.
    2. The mirror remote (GitHub) reachability check succeeds.
    3. Every configured critical queue (default: `mirror`, `watchlog`; configurable via policy) satisfies both queue health conditions below.
  - **Mirror remote check:** perform an asynchronous HTTP GET to a configurable endpoint (`MIRROR_HEALTH_ENDPOINT`, default `https://api.github.com/`) with a 2s timeout. Only 2xx responses are considered healthy. Results are cached for 30s; `/readyz` serves cached status to avoid blocking.
  - **Queue Thresholds (per queue):**
    - `pending_count <= 100` **AND** `success_rate >= 0.95` must both hold for the last 5-minute sliding window.
    - Sliding window: evaluate every minute using job event timestamps; include jobs with `enqueued_at` ≥ `now-5m` and < `now` (monotonic clock skew allowance ±5s).
    - `pending_count` counts jobs enqueued within the window that do not have a completion timestamp ≤ window end.
    - `processed_count` counts jobs completed within the window (regardless of enqueue time).
    - `success_rate = processed_count / (processed_count + pending_count)`; if denominator is zero, treat success_rate as 1.0 (healthy).
  - Critical queues default to `mirror` and `watchlog`, configurable via policy (`policy.stargate.critical_queues`). All configured queues must pass the checks. If the list is empty, queues are treated as healthy by default.
  - The endpoint returns HTTP 503 when any queue fails either condition, with a JSON body describing all unhealthy queues.
  - Responses must include `Content-Type: application/json` and `Date` headers. Numeric fields (`currentDepth`, `threshold`) are integers; timestamps are ISO8601 UTC.
  - Example responses:
    - 200 OK:
      ```json
      {"status":"ok","timestamp":"2025-10-26T12:34:56Z"}
      ```
    - 503 Service Unavailable:
      ```json
      {"status":"unhealthy","unhealthyQueues":[{"name":"mirror","currentDepth":150,"threshold":100,"firstObservedAt":"2025-10-26T12:30:00Z","lastObservedAt":"2025-10-26T12:34:56Z","details":"pending count over threshold"}],"timestamp":"2025-10-26T12:34:56Z","suggestedRemediation":"Investigate mirror worker backlog"}
      ```

## 3. Test Plan

- **Unit Test:** Test the logic for each health check condition in isolation.
- **Integration Test (`/healthz`):** Start the Stargate service. Verify `/healthz` returns 200. Simulate a repository access failure and verify it returns 500.
- **Integration Test (`/readyz`):** Start two Stargate instances. Verify the leader's `/readyz` returns 200. Verify the passive node's `/readyz` returns 503. Simulate a GitHub connectivity issue on the leader and verify its `/readyz` returns 503 with JSON error payload.
- **Integration Test (`/readyz` GitHub Timeout):** Simulate the mirror endpoint hanging for 60s; assert `/readyz` returns 503 within timeout and logs timeout.
- **Integration Test (Queue Threshold Crossing):** Increase queue depth above 100, assert `/readyz` returns 503; reduce below threshold and confirm return to 200; test success rate boundary at exactly 95%.
- **Integration Test (Leader Election Edge Cases):** Simulate split-brain (two leaders) and leader failover, verifying `/readyz` responses (503 for non-leader, 200 only for active leader).
- **Load Test (Concurrency):** Hit `/readyz` with 1000 concurrent requests; verify latency remains acceptable and responses consistent.
- **Integration Test (Success-Rate Boundaries):** Validate behavior at 94.9%, 95%, 95.1% success rate.
- **Integration Test (Clock Skew):** Simulate clock drift impacting 5-minute window calculations and ensure accurate readiness decisions.
