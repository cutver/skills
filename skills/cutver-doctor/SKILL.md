---
name: cutver-doctor
description: Inspect and verify version consistency across manifests, detect configuration issues, and check changelog drift.
triggers:
  - doctor
  - cutver doctor
  - verify manifests
  - check changelog drift
  - release health check
---

# cutver-doctor

Guide for running diagnostic health checks on workspace manifests, configuration files, and changelogs using `cutver doctor`.

## Overview

In multi-package, polyglot, or workspace setups (such as projects containing Rust `Cargo.toml`, Node `package.json`, Python `pyproject.toml`, or Tauri configs), version drift easily occurs when one manifest is bumped while others lag behind.

`cutver doctor` analyzes all configured manifests, reads their current declared versions, evaluates configuration correctness, and verifies that the changelog reflects the latest release state.

---

## Health Check Procedure

### Standard Diagnostic Check
Run `cutver doctor` from the project root to verify manifest synchronization and configuration validity:

```bash
cutver doctor
```

### Full Check (Including Changelog Drift)
To also verify that `CHANGELOG.md` is synchronized with the latest release tag and declared version:

```bash
cutver doctor --check-changelog
```

Use this check prior to cutting a release or in CI pipelines to prevent broken or desynchronized releases.

---

## Exit Codes & Semantics

`cutver doctor` utilizes deterministic exit codes suitable for CI/CD checks and agent decision branches:

| Exit Code | Status | Meaning |
|:---:|:---|:---|
| **`0`** | **Synchronized / Healthy** | All manifests share the exact same version, `cutver.toml` is valid, and changelog matches (when `--check-changelog` is passed). |
| **`1`** | **Configuration Error** | `cutver.toml` is missing, malformed, contains invalid syntax, or references non-existent files. |
| **`2`** | **Version Drift Detected** | Different manifests declare conflicting version numbers, or manifest versions do not match the expected state. |

---

## Troubleshooting & Remediation Guidance

### Remediation for Exit Code 1 (Configuration Error)
- **Symptom**: `cutver.toml` not found or invalid TOML syntax.
- **Remediation**:
  1. If `cutver.toml` is missing, initialize it by running `cutver init` (see `cutver-init` skill).
  2. If the configuration is invalid, inspect `cutver.toml` for syntax errors or invalid paths:
     ```bash
     cat cutver.toml
     ```
  3. Ensure all paths listed under `[workspace]` or `manifests` actually exist in the repository.

### Remediation for Exit Code 2 (Version Drift)
- **Symptom**: For example, `Cargo.toml` is at `1.2.0`, while `package.json` is at `1.1.9`.
- **Remediation**:
  1. Review the output of `cutver doctor` to identify the divergent files:
     ```text
     [DRIFT] Manifest version mismatch:
       - Cargo.toml: 1.2.0
       - package.json: 1.1.9
     ```
  2. Determine the canonical target version.
  3. Align the lagging manifests manually or run an explicit sync/bump:
     - To align files manually, edit the lagging manifest to match the canonical version.
     - Alternatively, re-run `cutver doctor` to confirm resolution after manual edit.

### Remediation for Changelog Drift (`--check-changelog`)
- **Symptom**: The changelog has no section header corresponding to the latest release version, or commit entries are missing.
- **Remediation**:
  1. Inspect the latest changelog header:
     ```bash
     head -n 25 CHANGELOG.md
     ```
  2. Generate missing changelog notes using `cutver changelog` or simulate a bump with `cutver bump auto --dry-run` to see what entries should be present.
