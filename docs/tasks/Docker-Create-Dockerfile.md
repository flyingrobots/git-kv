# Task: Create Dockerfile

**Feature:** `Dockerized Development Environment`

## 1. Description

Create a `Dockerfile` at the root of the repository. This file will define the base image and necessary tools for the development environment, supporting both local development (bind-mounted source) and CI jobs.

## 2. Acceptance Criteria

- A `Dockerfile` exists at `/Dockerfile`.
- The `Dockerfile` defines `ARG` values (e.g., `GO_VERSION=1.24`, `DEV_USER=developer`) and uses them in both the `FROM` statement (`golang:${GO_VERSION}` with optional digest pinning) and user-creation steps so CI or local developers can override defaults.
- Install the following tools, each justified for development parity:
  - `git` for fetching Go modules and other repositories.
  - `make` for running project make targets used in CI.
  - `ca-certificates` to enable TLS verification during module and asset downloads.
  - `gcc`/`build-essential` to provide the C/C++ toolchain necessary for cgo and native dependencies.
  - `curl` or `wget` for downloading artifacts and performing health checks.
  - `openssh-client` to support Git operations over SSH.
  - `bash` to ensure shell scripts run with expected features.
- Create a dedicated non-root user (default `developer` via `ARG DEV_USER`) and ensure it owns the working directory so bind-mounted source remains writable.
- Set `WORKDIR /work` and document (via comment or README link) that local development should mount the repository into `/work`.
- Omit `EXPOSE` directives entirely; this development container runs CLI tooling only and does not serve network traffic that needs published ports.
- Include a comment near the top of the Dockerfile clarifying the image is intended for local development and CI tasks.

## 3. Test Plan

- **Manual Test:** Build the Docker image locally (`docker build . -t git-kv-dev`).
- **Manual Test:** Rebuild with an overridden Go version (`docker build . -t git-kv-dev-1-24-3 --build-arg GO_VERSION=1.24.3`); confirm the resulting image reports the requested version in `go version`.
- **Manual Test:** Run a container from the image (`docker run --rm -it -v $(pwd):/work git-kv-dev bash`) and verify the process runs as the non-root user (`id -u` is not `0`).
- **Manual Test:** Inside the running container, touch a file under `/work` and verify it appears on the host, demonstrating writable bind mounts.
- **Manual Test:** Within the running container, run each of the following commands and expect a zero exit code plus a version banner or success message:
  - `go version` (should report `go version go1.24.x` reflecting overridden `GO_VERSION`).
  - `git --version` (should print the Git client version).
  - `make --version` (should print GNU Make version information).
  - `gcc --version` (confirms the C toolchain is available).
  - `curl --version` (or `wget --version` if that tool was installed) showing SSL support.
  - `ssh -V` (from `openssh-client`, should print OpenSSH release).
  - `bash --version` (ensures Bash is installed for scripts).
  - `curl https://example.com -I` (should succeed with HTTP 200/301, proving CA certificates allow TLS connections).

## 4. Implementation Guidance

- **Run as Non-Root:** Create a dedicated user via `ARG DEV_USER=developer`, switch to it with `USER $DEV_USER`, and `chown -R` the working directory so bind-mounted source stays writable.
- **Working Directory & Volumes:** Set `WORKDIR /work` and include comments reminding developers to mount the repository with `-v $(pwd):/work` for local editing.
- **Image Size & Multi-Stage Builds:** Consider a multi-stage workflow where the base stage (with build tooling) is reused for CI and a slimmer runtime stage is produced for deployments; document how to extend the dev image if additional tooling is required.
- **Configurable Tool Versions:** Provide `ARG`s (e.g., `GO_VERSION`, `DEV_USER`, optional `NODE_VERSION` or others when needed) so CI can override versions without editing the Dockerfile. Demonstrate usage in the test plan build commands.
- **Intended Usage Commentary:** Add inline comments or README references clarifying that the image targets local development and CI jobs, not production workloads, and that production images should strip dev-only tooling.
