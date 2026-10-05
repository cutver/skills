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
- Use `cutver/setup@v1` for binary installation in workflows; specify `cache: "false"` when fresh binary releases are required.
- Use `cutver/release@v1` with `workflow_dispatch` for on-demand control; never auto-release on arbitrary push to main without explicit approval gates.
- Consume native action outputs: `steps.cutver.outputs.released`, `steps.cutver.outputs.tag`, and `steps.cutver.outputs.version`.
- Authenticate release distribution using GitHub App token (`cutver-release[bot]`) via `actions/create-github-app-token@v3` to bypass push protections and trigger downstream workflows.

## Decision Gates
1. **Workflow Purpose**:
   - Tool installation / CI verification: configure `cutver/setup@v1` in lint/test workflows.
   - Release orchestration: configure `cutver/release@v1` in `.github/workflows/release.yml`.
2. **Release Trigger**:
   - `workflow_dispatch` input: pass `level` (`auto`, `patch`, `minor`, `major`) and `dry_run` boolean.
3. **Execution Mode**:
   - Dry run: test release calculation without creating tags or pushing commits.
   - Live release: perform mutating bump and git push.

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
3. Add Cutver release step:
   ```yaml
   - uses: cutver/release@v1
     id: cutver
     with:
       token: ${{ steps.app-token.outputs.token }}
       level: ${{ github.event.inputs.level || 'auto' }}
       dry-run: ${{ github.event.inputs.dry_run || false }}
   ```
4. Wire downstream release jobs consuming outputs (`released`, `tag`, `version`).

## Output Contract
- Generate valid YAML workflow file adhering to GitHub Actions schema.
- Confirm integration of `cutver/setup@v1` or `cutver/release@v1`.
- Verify secrets and token routing configuration.

## References
- Setup Action: `cutver/setup@v1`
- Release Action: `cutver/release@v1`
- Action outputs: `released` (boolean), `tag` (string), `version` (string)
