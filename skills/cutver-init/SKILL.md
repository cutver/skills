---
name: cutver-init
description: "Trigger: cutver init, init cutver, onboard cutver, scaffold cutver, configure cutver. Autodiscover workspace manifests and generate canonical cutver.toml configuration."
license: Apache-2.0
metadata:
  author: cutver
  version: "2.0"
---

## Activation Contract
Activate when the user asks to initialize, scaffold, onboard, or configure Cutver in a workspace, or runs commands matching `cutver init`.

## Hard Rules
- Always run discovery from the workspace repository root.
- Never overwrite an existing `cutver.toml` without explicit user confirmation.
- Respect monorepo workspace configurations; never configure subprojects as separate roots unless explicitly requested.
- Ensure all discovered manifest paths exist before generating configuration.

## Decision Gates
1. **Workspace Layout Detection**:
   - Single package: detect `Cargo.toml`, `package.json`, `pyproject.toml`, or `build.gradle*` at root.
   - Monorepo: detect workspace members across Cargo (`[workspace.members]`), npm/pnpm/yarn/bun (`workspaces` in `package.json` or `pnpm-workspace.yaml`), Gradle (`settings.gradle*`), or Python uv/poetry workspaces.
2. **Configuration Mode**:
   - Conventional mode (default): standard keep-a-changelog with conventional commit parsing.
   - Template mode: custom MiniJinja template referenced via `[changelog.template]`.

## Execution Steps
1. Scan the repository root and subdirectories to identify manifests (`Cargo.toml`, `package.json`, `pyproject.toml`, `build.gradle*`, `tauri.conf.json`).
2. Verify if `cutver.toml` already exists:
   - If present: stop and request explicit user confirmation before overwriting or use `cutver init --update`.
   - If absent: proceed with initialization.
3. Execute initialization command:
   ```bash
   cutver init
   ```
   For non-interactive environments, append `--yes` or appropriate flags.
4. Verify the generated `cutver.toml` by running:
   ```bash
   cutver doctor
   ```

## Output Contract
- Report detected manifest types and paths.
- Display generated `cutver.toml` summary (manifest list, changelog mode, git tag settings).
- State doctor verification status (`cutver doctor` exit code 0 confirmation).

## References
- `cutver.toml` specification: `https://github.com/cutver/cutver#configuration`
- Manifest types: Cargo (`cargo-package`), JSON (`json`), TOML/pyproject (`pyproject`), Gradle (`gradle`).
