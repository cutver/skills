---
name: cutver-init
description: Initialize and onboard projects to Cutver by autodiscovering multi-language manifests and generating cutver.toml.
triggers:
  - init
  - cutver init
  - configure cutver
  - setup cutver
  - onboard cutver
---

# cutver-init

Guide for onboarding workspaces and repositories to Cutver, autodiscovering manifests across ecosystems, and configuring `cutver.toml`.

## Overview

`cutver init` scans your repository root and subdirectories to automatically detect supported project manifests:
- **Rust**: `Cargo.toml` (single crate and multi-crate Cargo workspaces)
- **Node.js / JavaScript / TypeScript**: `package.json` (npm, pnpm, yarn, bun workspaces)
- **Python**: `pyproject.toml` (Poetry, Flit, Hatch, uv)
- **JVM / Android**: `build.gradle`, `build.gradle.kts`, `gradle.properties`
- **Tauri**: `src-tauri/tauri.conf.json`

It aggregates the discovered manifests and generates a tailored `cutver.toml` configuration file.

---

## Onboarding Procedure

### Step 1: Inspect Workspace Structure
Before running initialization, inspect the repository to understand its layout:

```bash
# Check existing manifest files
git ls-files "*Cargo.toml" "*package.json" "*pyproject.toml" "*tauri.conf.json" "*gradle*"
```

### Step 2: Run `cutver init`
Execute `cutver init` at the root of the repository:

```bash
cutver init
```

Cutver will:
1. Scan for known manifest types across the repository tree.
2. Detect the current version across discovered files.
3. Suggest a baseline configuration and prompt for confirmation (or generate automatically in non-interactive modes).
4. Write the resulting `cutver.toml`.

### Step 3: Understanding the Generated `cutver.toml`
A typical generated `cutver.toml` declares manifests and release policies:

```toml
# cutver.toml - release orchestration configuration

[[manifest]]
path = "Cargo.toml"
kind = "cargo-package"

[[manifest]]
path = "package.json"
kind = "json"
field = "version"

[[manifest]]
path = "pyproject.toml"
kind = "pyproject"

[changelog]
path = "CHANGELOG.md"
format = "keep-a-changelog"
mode = "conventional"

[git]
tag_prefix = "v"
commit_message = "chore(release): v{version} [skip ci]"
require_clean_tree = true
require_branch = "main"

[publish]
push = true
```

### Step 4: Verify the New Configuration
Immediately verify the configuration using `cutver doctor`:

```bash
cutver doctor
```

If doctor exits with `0`, onboarding is complete and the repository is ready for releases.

---

## Updating Existing Configurations (`--update`)

When adding new sub-crates, packages, or services to an already configured repository:

```bash
# Scan repository for new manifests and append them to cutver.toml
cutver init --update
```

The `--update` flag:
- Preserves your existing custom settings (git commit template, changelog format, tag prefix).
- Detects newly created manifests not yet tracked in `cutver.toml`.
- Merges the newly discovered paths into the `manifests` array.

---

## Command Flags & Customization

### Flags
- `--update`: Updates an existing `cutver.toml` with newly detected manifests without overwriting user settings.
- `--force`: Overwrites an existing `cutver.toml` file with a freshly generated configuration.
- `--manifest <TYPE>`: Restrict manifest discovery to specific ecosystems (e.g. `cargo`, `npm`, `python`).

### Common Customizations in `cutver.toml`
- **`tag_prefix`**: Set to `""` if tags should be `1.0.0` instead of `v1.0.0`.
- **`git.commit_message`**: Customize conventional commit format (e.g., `release: v{{ version }}`).
- **`manifests`**: Add custom JSON, TOML, or YAML files using JSONPath or regex selectors if using non-standard file formats.
