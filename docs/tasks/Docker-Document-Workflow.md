# Task: Document Docker Development Workflow

**Feature:** `Dockerized Development Environment`

## 1. Description

Add clear instructions to the `README.md` file explaining how to set up and use the Dockerized development environment.

## 2. Acceptance Criteria

- The `README.md` contains a new section titled "Dockerized Development Environment" or similar.
- This section provides step-by-step instructions on:
  - Build the Docker image.
  - Start the development container.
  - Access a shell inside the container.
  - Run tests and linters from the host or within the container.
  - Stop and remove the container.
- The documentation emphasizes the benefits of using the Dockerized environment.
Environment baseline: macOS 14 (or Ubuntu 22.04) with Docker Desktop 24.0+ (or Docker Engine 24.0+) installed and running; Git and Make available on the host.

- **Automated Smoke Script (optional but recommended):** From repo root run `./scripts/verify-docker-workflow.sh`. Expect exit code `0` and summary `WORKFLOW OK`. Failing exit code indicates test failure.
- **Manual Test Sequence:** Execute the following steps in a fresh terminal; each step must finish within 2 minutes unless noted. All commands should exit `0`.
  1. `docker build . -f Dockerfile -t git-kv-dev-docs` — Expect build success and image listed via `docker images git-kv-dev-docs`.
  2. `docker run --rm -d --name git-kv-docs -v $(pwd):/work git-kv-dev-docs tail -f /dev/null` — Container should transition to `healthy` or `running` within 60s (`docker ps --filter name=git-kv-docs`).
  3. `docker exec git-kv-docs id -u` — Output must be non-zero (verifies non-root user).
  4. `docker exec git-kv-docs make test` — Should succeed (or document expected placeholder output). Capture logs and confirm they contain `PASS` or equivalent.
  5. `docker exec git-kv-docs make lint` — If lint target exists; otherwise confirm README states how to skip. Expect `0` exit code.
  6. `docker logs git-kv-docs | tail -n5` — Logs should include line `Ready for development` (or whatever README instructs). Missing string is a failure.
  7. `docker exec git-kv-docs curl -s -o /dev/null -w "%{http_code}" http://localhost:8080/health` (only if README advertises a health endpoint). Expect `200`; if endpoint is optional ensure README documents that.
  8. `docker stop git-kv-docs` — Should stop within 30s.
  9. `docker container prune -f` — Cleanup step; README must mention cleanup.

PASS/FAIL rule: All steps must meet their expected exit codes, timing, and log output. Any deviation is a FAIL and must be addressed before merging.
