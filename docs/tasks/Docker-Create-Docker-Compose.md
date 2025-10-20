# Task: Create `docker-compose.yml`

**Feature:** `Dockerized Development Environment`

## 1. Description

Create a `docker-compose.yml` file to orchestrate the development container. This file will define the service, build context, and volume mounts.

## 2. Acceptance Criteria

- A `docker-compose.yml` file exists at `/docker-compose.yml`.
- It defines a service (e.g., `dev`) that builds from the `Dockerfile`.
- It mounts the local project directory into the container's working directory (e.g., `/work`).
- The service is configured to keep the container running indefinitely (e.g., `command: tail -f /dev/null`).
- The service is named descriptively.

## 3. Test Plan

- **Manual Test:** Run `docker-compose up -d`. Verify the container starts and stays running.
- **Manual Test:** Run `docker-compose exec dev bash`. Verify you can access the container's shell.
- **Manual Test:** Make a change to a local file. Verify the change is immediately visible inside the container's mounted volume.
