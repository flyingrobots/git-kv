# Task: Stargate Health Endpoints

**Feature:** `US-G10 — HA Failover`

## 1. Description

Implement standard HTTP health endpoints (`/healthz` and `/readyz`) for the Stargate service. These endpoints are crucial for load balancers, orchestrators (like Kubernetes), and monitoring systems to determine the health and readiness of Stargate instances.

## 2. Acceptance Criteria

- The Stargate service exposes an HTTP server (e.g., on a configurable port).
- A `/healthz` endpoint is available.
  - It returns HTTP 200 OK if the Stargate process is running and basic dependencies (e.g., Git repository access) are functional.
  - It returns HTTP 500 Internal Server Error otherwise.
- A `/readyz` endpoint is available.
  - It returns HTTP 200 OK only if the Stargate instance is the elected leader AND its mirror remote (GitHub) is reachable AND its internal queues are draining.
  - It returns HTTP 503 Service Unavailable otherwise.

## 3. Test Plan

- **Unit Test:** Test the logic for each health check condition in isolation.
- **Integration Test (`/healthz`):** Start the Stargate service. Verify `/healthz` returns 200. Simulate a repository access failure and verify it returns 500.
- **Integration Test (`/readyz`):** Start two Stargate instances. Verify the leader's `/readyz` returns 200. Verify the passive node's `/readyz` returns 503. Simulate a GitHub connectivity issue on the leader and verify its `/readyz` returns 503.
