---
name: cutver-release
description: "Trigger: cutver bump, cutver release, cut release, bump version, semver bump, first release. Execute safe semantic version bumps with mandatory dry-run simulation and health checks."
license: Apache-2.0
metadata:
  author: cutver
  version: "2.0"
---

## Activation Contract
Activate when the user requests a version bump, cutting a release, creating a new tag, or running `cutver bump`/`cutver release`.

## Hard Rules
- GOLDEN RULE: ALWAYS execute `cutver bump auto --dry-run` (or explicit level with `--dry-run`) FIRST before mutating files or creating git commits/tags.
- Verify clean working tree (`git status --porcelain` must be empty) before applying any mutating release.
- Use `--first-release` for initial project release without incrementing version.
- Ensure `floating_major_tag` is respected when configured.
- Never force git pushes or bypass doctor verification.

## Decision Gates
1. **Bump Level Selection**:
   - `auto` (default): deduct semver increment (`patch`, `minor`, `major`) automatically from conventional commits.
   - Explicit level: user specified `patch`, `minor`, or `major`.
2. **First Release vs Standard Bump**:
   - Initial release: pass `--first-release` to tag current version without bumping numbers.
   - Standard release: calculate increment from previous tag.
3. **Dry-Run Confirmation**:
   - Simulation mode (`--dry-run`): parse output and present planned changes to user.
   - Execution mode: execute mutating bump only after plan confirmation.

## Execution Steps
1. Verify git working tree is clean:
   ```bash
   git status --porcelain
   ```
   Halt if uncommitted changes exist.
2. Run diagnostic health check:
   ```bash
   cutver doctor --check-changelog
   ```
   Halt if exit code is non-zero.
3. Simulate bump in dry-run mode:
   ```bash
   cutver bump auto --dry-run
   ```
   (Replace `auto` with `patch`/`minor`/`major` or add `--first-release` as selected).
4. Present simulated plan to user: current version, target version, affected manifests, commit summary.
5. Upon user confirmation, execute the mutating bump:
   ```bash
   cutver bump auto
   ```
6. Verify resulting commit and tag:
   ```bash
   git show --stat HEAD
   cutver doctor
   ```

## Output Contract
- Dry-run preview summary: old version, new version, bump type, manifests to update.
- Mutated state summary: updated files, generated commit hash, created tag name, push status.

## References
- CLI Bump command: `cutver bump [auto|patch|minor|major] [--dry-run] [--first-release]`
- Preflight doctor check: `cutver doctor --check-changelog`
