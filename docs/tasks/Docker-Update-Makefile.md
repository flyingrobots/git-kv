# Task: Update Makefile for Docker Environment

**Feature:** `Dockerized Development Environment`

## 1. Description

Modify the existing `Makefile` so that `make test` and `make lint` detect whether they are running inside the Docker development container. When the `DEV_INSIDE_CONTAINER` environment variable is set (or, as a fallback, when `/.dockerenv` exists), the targets should run the underlying commands directly. When executing on the host, the targets must delegate to `docker-compose exec <service> <command>` without attempting to auto-start the compose stack. Provide dedicated `make docker-up` and `make docker-down` helpers for developers to manage the lifecycle (`docker-compose up -d --build` and `docker-compose down --volumes --remove-orphans`). Document that the container provides a `bash` shell and a PATH including project binaries (e.g., `/usr/local/go/bin:/work/bin:$PATH`).

## Prerequisites

- Docker Compose service name: `dev` (targets should reference this service).
- Docker Compose version: v2.20+ (supports `docker compose` and `docker-compose` invocations; prefer `docker-compose`).
- In-container detection: the environment variable `DEV_INSIDE_CONTAINER=1` will be exported by the Docker image; if absent, targets may fall back to checking for the file `/.dockerenv`.

## 2. Acceptance Criteria

- `make test` and `make lint` inspect `DEV_INSIDE_CONTAINER` (or `/.dockerenv`) to determine execution context; on the host they shell out to `docker-compose exec dev <cmd>`, and inside the container they run the commands directly.
- The updated targets honour the existing environment (no implicit `docker-compose up`) and document that developers should run `make docker-up` first.
- `make test` and `make lint` succeed when invoked inside the container with `DEV_INSIDE_CONTAINER=1` exported.
- A new `make docker-up` target starts the `dev` service via `docker-compose up -d --build`.
- A new `make docker-down` target stops and removes the stack via `docker-compose down --volumes --remove-orphans`.
- Documentation/comments in the `Makefile` state that container sessions provide `bash` and a PATH including project binaries.

## 3. Test Plan

- **Manual Test (Host happy-path):**
  1. `make docker-up` — Expect exit code `0`. Verify `docker ps --filter name=git-kv-dev --format '{{.Names}}'` prints the service container name (e.g., `git-kv-dev`).
  2. `make test` — Expect exit code `0`. Capture output; ensure it contains `All tests passed` (or project-specific success string) and confirm command executed inside container by checking `docker logs git-kv-dev | tail -n20` includes the test invocation.
  3. `make lint` — Expect exit code `0`. Capture output; ensure it contains the lint summary (e.g., `Lint completed`). Confirm container execution via logs as above.

- **Manual Test (In-container execution):**
  1. `docker-compose exec dev bash` to open a shell; verify `echo $DEV_INSIDE_CONTAINER` prints `1` and `/work` is writable (`touch /work/.probe` and ensure no error).
  2. Run `make test` and `make lint` from this shell. Expect exit code `0` for both, no Docker socket errors, and standard output indicating success.

- **Manual Test (Teardown):** `make docker-down` — Expect exit code `0`. Verify `docker ps --filter name=git-kv-dev` returns empty output and `docker volume ls --filter name=git-kv` (or project-specific volumes) shows no dangling volumes. Confirm `docker network ls --filter name=git-kv` reports nothing unexpected.

- **Failure scenarios:**
  - Stop the container manually (`docker stop git-kv-dev`) and run `make test`; expect a non-zero exit code and a clear error message instructing to run `make docker-up`.
  - Unset `DEV_INSIDE_CONTAINER` inside the container and rerun `make test`; expect detection fallback (`/.dockerenv`) still runs tests directly.

- **Cleanup verification:** After tests, ensure `rm /work/.probe` succeeds and no temporary artifacts remain. Confirm `ps aux | grep docker-compose` shows no lingering compose processes.
