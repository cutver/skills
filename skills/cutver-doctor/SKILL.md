---
name: cutver-doctor
description: "Trigger: cutver doctor, doctor, check changelog drift, verify manifests, release health check. Diagnose manifest synchronization, configuration health, and changelog drift."
license: Apache-2.0
metadata:
  author: cutver
  version: "2.0"
---

## Activation Contract
Activate when the user asks to run diagnostics, verify manifests, check for version drift, validate changelog alignment, or runs `cutver doctor`.

## Hard Rules
- Always run `cutver doctor` before attempting any mutating release bump.
- Interpret process exit codes deterministically:
  - `0`: Workspace healthy, manifests synchronized, changelog aligned.
  - `1`: Configuration error (`cutver.toml` missing, malformed, or references missing path).
  - `2`: Version drift detected across manifests or changelog out of sync.
- Always include `--check-changelog` during pre-release validation checks.

## Decision Gates
1. **Exit Code Evaluation**:
   - `0`: Proceed to next operational phase (release bump or pipeline completion).
   - `1`: Halt execution; repair syntax or paths in `cutver.toml` or execute `cutver init`.
   - `2`: Halt execution; identify divergent manifests or missing changelog entries and synchronize versions.

## Execution Steps
1. Run diagnostic command from the workspace repository root:
   ```bash
   cutver doctor --check-changelog
   ```
2. Capture standard output, standard error, and exit status code.
3. If exit code is `1`:
   - Inspect `cutver.toml` for syntax errors or nonexistent paths.
   - Prompt user or execute repair.
4. If exit code is `2`:
   - Extract mismatched manifest versions and paths from doctor output.
   - If version drift: align divergent manifests to canonical target version.
   - If changelog drift: inspect `CHANGELOG.md` header against latest release tag.
5. Re-run `cutver doctor --check-changelog` to confirm exit code `0`.

## Output Contract
- Emit clear status block: Health status (Healthy | Config Error | Version Drift).
- Detail any divergent manifest paths and their respective declared versions.
- Provide actionable next steps or automated remediation status.

## References
- Exit code semantics: `0` (clean), `1` (config error), `2` (drift).
- Verification command: `cutver doctor --check-changelog`
