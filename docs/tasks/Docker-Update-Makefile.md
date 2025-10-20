# Task: Update Makefile for Docker Environment

**Feature:** `Dockerized Development Environment`

## 1. Description

Modify the existing `Makefile` to ensure that `make test` and `make lint` commands can be executed both directly on the host and, more importantly, from within the Docker development container.

## 2. Acceptance Criteria

- The `Makefile` is updated to use `docker-compose exec` for `test` and `lint` targets when run from the host.
- The `Makefile` targets (e.g., `test`, `lint`) function correctly when executed inside the Docker container.
- A new `make docker-up` target is added to start the development container.
- A new `make docker-down` target is added to stop and remove the development container.

## 3. Test Plan

- **Manual Test (Host):** Run `make docker-up`. Then run `make test` and `make lint` from the host. Verify they execute inside the container.
- **Manual Test (Container):** Run `docker-compose exec dev bash`, then run `make test` and `make lint` from inside the container. Verify they execute correctly.
- **Manual Test:** Run `make docker-down` and verify the container is stopped and removed.
