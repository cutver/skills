---
name: cutver-actions
description: "Trigger: cutver action, cutver/setup, cutver/release, github actions cutver, ci release workflow. Scaffold and configure GitHub Actions workflows for Cutver setup and automated release orchestration."
license: Apache-2.0
metadata:
  author: cutver
  version: "2.0"
---

## Activation Contract
Activate when the user asks to configure GitHub Actions CI/CD workflows, install Cutver in GitHub runners, or automate releases with `cutver/setup` or `cutver/release`.

## Hard Rules
- Use `cutver/setup@v1` for binary installation in workflows. The integration guide documents a `github-token` input and states the action downloads the release archive with no checksum or signature check.
- Use `cutver/release@v1` with `workflow_dispatch` for on-demand control. A push-to-`main` trigger is possible and the integration guide shows one, so wire it only when the user asks for it and keep the release behind their approval.
- Consume the documented `cutver/release@v1` outputs: `released`, `version`, `tag`, `notes-path`, and `release-notes`, all `string`.
- Authenticate release distribution with a GitHub App token (`cutver-release[bot]`) minted by `actions/create-github-app-token@v3`: it is what attributes the bump commit and the release to the bot and what lets a push through a ruleset that lists that app as a bypass actor. Whether a given repository's ruleset lists it is that repository's setting, not something the action guarantees.

## Decision Gates
1. **Workflow Purpose**:
   - Tool installation / CI verification: configure `cutver/setup@v1` in lint/test workflows.
   - Release orchestration: configure `cutver/release@v1` in `.github/workflows/release.yml`.
2. **Release Trigger**:
   - `workflow_dispatch` with caller-defined inputs for the bump level, the first release, and the dry run; `cutver/release@v1` is documented with the `command`, `bump`, `template`, `first-release`, and `dry-run` inputs.
3. **Execution Mode**:
   - Preview: pass `dry-run` to `cutver/release@v1`, or run `cutver bump auto --dry-run` as its own step.
   - Live: omit `dry-run` and let `cutver/release@v1` run the release lifecycle.

## Execution Steps
1. Create or edit `.github/workflows/release.yml`.
2. Configure token authentication step:
   ```yaml
   - uses: actions/create-github-app-token@v3
     id: app-token
     with:
       app-id: ${{ secrets.RELEASE_APP_ID }}
       private-key: ${{ secrets.RELEASE_APP_PRIVATE_KEY }}
   ```
3. Add Cutver release step, using the inputs the released guides document:
   ```yaml
   - uses: cutver/release@v1
     id: cutver
     with:
       command: release
       bump: ${{ github.event.inputs.bump }}
       first-release: ${{ github.event.inputs.first_release }}
       dry-run: ${{ github.event.inputs.dry_run }}
     env:
       GITHUB_TOKEN: ${{ steps.app-token.outputs.token }}
   ```
4. Wire downstream release jobs consuming the `released`, `version`, `tag`, and `release-notes` outputs.

## Output Contract
- Generate valid YAML workflow file adhering to GitHub Actions schema.
- Confirm integration of `cutver/setup@v1` or `cutver/release@v1`.
- Verify secrets and token routing configuration.

## References
- Setup Action: `cutver/setup@v1` (`github-token` input; no checksum or signature check of the downloaded archive)
- Release Action: `cutver/release@v1` (documented inputs: `command`, `bump`, `first-release`, `dry-run` in the production release workflow guide; `template` in the GitHub Actions guide)
- Action outputs: `released` (`"true"` / `"false"` string), `version`, `tag`, `notes-path`, `release-notes`
- The actions' complete input and output sets live in `github.com/cutver/setup` and `github.com/cutver/release`, not in this repository.
