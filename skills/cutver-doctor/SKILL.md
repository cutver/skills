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
- Run `cutver doctor` before attempting any mutating release bump.
- Interpret exit codes from what `doctor` itself does:
  - `0`: No drift reported, and with `--check-changelog` no changelog/tag mismatch.
  - `1`: `doctor` could not complete: the configuration failed to load; a declared manifest other than a template sink cannot be read or holds an invalid version string; a declared plugin cannot be loaded (a wasm plugin in a build without the plugin runtime fails with `WasmNotSupported`) or a `kind = "plugin"` manifest cannot resolve its plugin; and, under `--check-changelog`, the changelog file cannot be read or `git tag -l` fails. The message names the cause.
  - `2`: `doctor` detected drift — a manifest version disagrees with the source of truth, a template sink's rendered output differs from the file on disk (a missing sink is read as empty, so a non-empty template reports drift rather than exit `1`), or `--check-changelog` found a changelog/tag mismatch.
- `doctor` reads the configuration and every declared manifest; the changelog file and `git tag -l` are read only with `--check-changelog`. It runs no preflight step, no hook, no publish command, and renders no changelog body, so `[changelog] format = "plugin"` is never consulted.
- Include `--check-changelog` during pre-release validation checks.

## Decision Gates
1. **Exit Code Evaluation**:
   - `0`: Proceed to next operational phase (release bump or pipeline completion).
   - `1`: Halt and read stderr: it names the configuration path, manifest path, plugin, or git command that failed. Repair that cause at the named path; `doctor` repairs nothing itself.
   - `2`: Halt execution; identify divergent manifests or missing changelog entries and synchronize versions. A template sink reported as `template output differs` is regenerated from its template, not hand-edited.

## Execution Steps
1. Run diagnostic command from the workspace repository root:
   ```bash
   cutver doctor --check-changelog
   ```
2. Capture standard output, standard error, and exit status code.
3. If exit code is `1`:
   - Read the error message: it names the configuration path, manifest path, plugin, or git command that failed.
   - Repair that cause at the named path. `cutver init --update` is not a repair here: it only appends manifests whose path is not yet declared and otherwise prints `ℹ cutver.toml is already up to date (no new manifests discovered).` Regenerating with `cutver init --force` does rewrite a stale entry, at the cost of every manual setting, so confirm with the user first.
4. If exit code is `2`:
   - Extract mismatched manifest versions and paths from doctor output.
   - If version drift: align divergent manifests to canonical target version.
   - If changelog drift: inspect `CHANGELOG.md` header against latest release tag.
5. Re-run `cutver doctor --check-changelog` to confirm exit code `0`.

## Output Contract
- Emit clear status block: Health status (Healthy | Error | Version Drift).
- Detail any divergent manifest paths and their respective declared versions.
- Provide actionable next steps or automated remediation status.

## References
- Exit code semantics: `0` (clean), `1` (`doctor` could not complete; the message names the failing file, plugin, or git command), `2` (drift).
- Verification command: `cutver doctor --check-changelog`
