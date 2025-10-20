# Task: Create Dockerfile

**Feature:** `Dockerized Development Environment`

## 1. Description

Create a `Dockerfile` at the root of the repository. This file will define the base image and necessary tools for the development environment.

## 2. Acceptance Criteria

- A `Dockerfile` exists at `/Dockerfile`.
- The `Dockerfile` uses a recent, stable Go image (e.g., `golang:1.21`).
- Installs `git` and other essential command-line tools required for development.
- Sets up a working directory (e.g., `/work`).
- Optionally exposes service ports only if the container runs networked services or a UI; otherwise, no ports are required.

## 3. Test Plan

- **Manual Test:** Build the Docker image locally (`docker build . -t git-kv-dev`).
- **Manual Test:** Run a container from the image (`docker run --rm -it git-kv-dev bash`).
- **Manual Test:** Verify that `go version`, `git --version`, and any other installed tools are available and functional inside the container.
