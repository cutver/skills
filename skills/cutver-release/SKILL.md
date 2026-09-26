---
name: cutver-release
description: Safely bump versions, synchronize multi-language manifests, generate changelogs, and cut releases using Cutver.
triggers:
  - release
  - cut release
  - bump version
  - semver bump
  - cutver bump
  - cutver release
---

# cutver-release

Guide for executing safe, automated, or manual semantic version bumps and releases across multi-manifest repositories using `cutver`.

## Core Principle (The Golden Rule)

> **Golden Rule**: **ALWAYS** run `cutver bump auto --dry-run` (or explicit level with `--dry-run`) **FIRST** before mutating any files or creating git commits/tags.

Never execute a mutating version bump without first previewing the simulated changes, confirming version calculations, and presenting the proposed diff/plan to the user.

---

## Step-by-Step Release Procedure

Follow this strict step-by-step workflow whenever cutting a release or bumping versions:

### Step 1: Verify Clean Working Tree
Ensure the git working directory has no uncommitted changes or unstaged modifications.

```bash
git status --porcelain
```

- If changes are present, stop and ask the user to commit or stash them before proceeding.
- Ensure the current branch is the designated release branch (e.g. `main` or `master`) and up to date with remote:
  ```bash
  git pull --ff-only
  ```

### Step 2: Validate Manifest Health & Sync
Run `cutver doctor` to confirm that all tracked manifests and changelog files are healthy and synchronized before bumping.

```bash
cutver doctor --check-changelog
```

- If `cutver doctor` reports drift or errors (non-zero exit code), abort the release and remediate using the `cutver-doctor` skill first.

### Step 3: Simulate the Bump (Dry Run)
Execute the bump command with `--dry-run` to calculate the next semver increment from conventional commits and inspect affected manifests.

```bash
# Automated bump based on conventional commits
cutver bump auto --dry-run

# Or explicit bump level if specified by user
cutver bump patch --dry-run
cutver bump minor --dry-run
cutver bump major --dry-run
```

### Step 4: Present Planned Changes for Confirmation
Present the output of the dry-run to the user or reviewer:
- Current version vs. proposed target version.
- List of manifests that will be updated (e.g. `Cargo.toml`, `package.json`, `pyproject.toml`).
- Overview of parsed commit messages and release type (patch, minor, major).

Confirm that the calculated version and bump scope match intent.

### Step 5: Execute the Bump
Once confirmed, run the mutating bump command:

```bash
# Auto semver resolution
cutver bump auto

# Or explicit level
cutver bump patch
cutver bump minor
cutver bump major
```

Cutver will:
1. Run preflight checks (clean git status, valid config).
2. Bump version numbers across all configured manifests simultaneously.
3. Generate or update `CHANGELOG.md` with conventional commit entries.
4. Stage modified files, create a git commit, and create a signed/annotated git tag (if configured in `cutver.toml`).

### Step 6: Verify Changes and Generated Changelog
Inspect the resulting git commit and changelog entry:

```bash
# View the generated commit and diff
git show --stat HEAD
git log -1 -p CHANGELOG.md

# Verify health status post-bump
cutver doctor
```

Verify that all manifest versions are identical and the changelog correctly categorizes breaking changes, features, and fixes.

---

## Command Reference & Flags

### Subcommands
- `cutver bump auto`: Automatically detects the semver increment (patch, minor, major) by inspecting conventional commits since the last release tag.
- `cutver bump patch`: Forces a patch version bump (`x.y.Z+1`).
- `cutver bump minor`: Forces a minor version bump (`x.Y+1.0`).
- `cutver bump major`: Forces a major version bump (`X+1.0.0`).

### Common Flags
- `--dry-run`: Simulates the version calculation, manifest updates, and changelog generation without touching files or git history. **Always use before mutating.**
- `--skip-preflight`: Skips initial preflight validations (such as checking for an uncommitted git working tree). Use with caution, typically in synthetic or test environments.
- `-c, --config <PATH>`: Path to a custom `cutver.toml` configuration file (default: `./cutver.toml`).
- `-v, --verbose`: Enables verbose debug logging for diagnosing commit parsing or manifest discovery.
