---
name: cutver-actions
description: "Trigger: cutver action, cutver/setup, cutver/release, github actions cutver, ci release workflow. Wire Cutver into GitHub Actions: install the binary and cut a release with its commands."
license: Apache-2.0
metadata:
  author: cutver
  version: "3.0"
---

## Activation Contract
Activate when the user asks to configure GitHub Actions CI/CD workflows, install Cutver in GitHub runners, or release with Cutver from CI.

`cutver/setup` installs the binary. The release itself is driven by the `cutver` commands, in two phases. `cutver/release` is **deprecated**: still published and usable, no longer the recommended path — it runs whatever published `cutver` is on `PATH`, so it cannot cut a release that fixes the release path, and its command set has no `publish` and no `--defer-publish`.

## Hard Rules
- Install with `cutver/setup@v1`. The integration guide documents a `github-token` input and states the action downloads the release archive with no checksum or signature check.
- Never propose `cutver/release@v1` for a new workflow. If a repository already uses it, say it is deprecated and offer the command steps below instead.
- Cut the release with the binary built from the commit being released, in two invocations: `cutver bump … --defer-publish` then `cutver publish --tag … --commit …`. One invocation cannot build and smoke artifacts before tagging.
- Preview first: `--dry-run` on the bump, or on the closing phase. A release is irreversible once tagged.
- Authenticate with a GitHub App token (`cutver-release[bot]`) minted by `actions/create-github-app-token@v3`: it attributes the commit and the release to the bot, and it is what a ruleset can list as a bypass actor. Whether a given ruleset does is that repository's setting.

## Decision Gates
1. **Workflow Purpose**:
   - Install / verify only: `cutver/setup@v1` in a lint or test workflow.
   - Release: the command steps below, in `.github/workflows/release.yml`.
2. **Release Trigger**:
   - `workflow_dispatch` by default, with caller-defined inputs for the bump level and the dry run.
   - `push` to `main`: only when the user asks, and behind their approval.
3. **Phase Split**:
   - Prepare: bump, push the branch, write a report.
   - Publish: tag, push, and hand the notes over — after the artifact jobs and smokes succeed (`needs:`).

## Execution Steps
1. Install the binary in the job that will run the commands, and put it on `PATH`. Build it from the commit being released when the workflow cuts its own release; install a published tag only to verify what a user downloads.
2. Mint the app token:
   ```yaml
   - uses: actions/create-github-app-token@v3
     id: app-token
     with:
       app-id: ${{ secrets.RELEASE_APP_ID }}
       private-key: ${{ secrets.RELEASE_APP_PRIVATE_KEY }}
   ```
3. Prepare the release:
   ```bash
   cutver doctor --check-changelog
   cutver bump "${LEVEL:-auto}" --defer-publish --report .cutver-release.json
   cutver changelog latest --template .github/templates/cutver/RELEASE.md > RELEASE_NOTES.md
   ```
4. Build and smoke the artifacts from the commit the report names, in jobs that depend on the prepare job.
5. Publish from a job whose `needs:` covers those jobs:
   ```bash
   cat RELEASE_NOTES.md | cutver publish --tag "$TAG" --commit "$COMMIT" --notes-file -
   ```
6. Upload assets and promote the draft release, which the `on_post_release` plugin creates.

## Output Contract
- Produce a valid GitHub Actions workflow: prepare and publish in separate jobs, linked by `needs:` and by the reported `commit`.
- No `cutver/release@v1` step in a new workflow.
- Token scoped to the jobs that push; secrets never echoed.

## References
- Setup action: `cutver/setup@v1` (`github-token` input; the download is not checksum- or signature-verified by the action)
- Commands: `cutver bump --defer-publish`, `cutver publish --tag --commit --notes-file`
- Pipeline and ordering: `docs/internals/pipeline.md` in the Cutver repository
- Migration from the deprecated action: `docs/integrations/migrating-from-the-release-action.md` in the Cutver repository
