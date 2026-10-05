---
name: cutver-containers
description: "Trigger: cutver container, cutver docker, cutver podman, cutver wslc, ghcr.io/cutver/cutver. Run Cutver inside Docker, Podman, or WSL Containers with proper workspace mounts and permissions."
license: Apache-2.0
metadata:
  author: cutver
  version: "2.0"
---

## Activation Contract
Activate when the user asks to run Cutver via Docker, Podman, WSL Containers, or reference `ghcr.io/cutver/cutver`.

## Hard Rules
- Always mount the current workspace directory to `/workspace`: `-v "$PWD:/workspace"`.
- Adhere to official container image tag conventions: `ghcr.io/cutver/cutver:latest`, `:v<version>`, or major tag `:v0`.
- When running in non-root or SELinux environments, apply proper volume flags and git safe directory configurations.
- Default to simulation (`--dry-run`) or diagnostic (`doctor`) operations unless mutating execution is explicitly requested.

## Decision Gates
1. **Container Engine**:
   - Docker: `docker run --rm -v "$PWD:/workspace" -w /workspace ghcr.io/cutver/cutver <command>`.
   - Podman (rootless/SELinux): append `:Z` volume flag: `-v "$PWD:/workspace:Z"`.
   - WSL Containers (`wslc`): invoke via `wslc run ghcr.io/cutver/cutver <command>`.
2. **Subcommand**:
   - Diagnostic: `doctor --check-changelog`.
   - Release simulation: `bump auto --dry-run`.
   - Initialization: `init`.

## Execution Steps
1. Detect available container engine (`docker`, `podman`, or `wslc`).
2. Construct container execution command with workspace mount:
   - For Docker:
     ```bash
     docker run --rm -v "$PWD:/workspace" -w /workspace ghcr.io/cutver/cutver doctor
     ```
   - For Podman:
     ```bash
     podman run --rm -v "$PWD:/workspace:Z" -w /workspace ghcr.io/cutver/cutver doctor
     ```
3. For git operations inside container, configure safe directory if required:
   ```bash
   docker run --rm -v "$PWD:/workspace" -w /workspace \
     -e GIT_CONFIG_COUNT=1 -e GIT_CONFIG_KEY_0=safe.directory -e GIT_CONFIG_VALUE_0=/workspace \
     ghcr.io/cutver/cutver bump auto --dry-run
   ```
4. Execute command and return standard output to user.

## Output Contract
- Deliver container run invocation matching user container runtime.
- Stream execution logs and container exit codes.
- Report completion status of containerized Cutver task.

## References
- Container Registry: `ghcr.io/cutver/cutver`
- Tags: `latest`, `v0`, `v0.10.0`
