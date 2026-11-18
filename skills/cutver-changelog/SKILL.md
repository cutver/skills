---
name: cutver-changelog
description: "Trigger: cutver changelog, changelog latest, cutver changelog show, extract release notes, changelog template. Extract release notes and render changelog templates for releases and PRs."
license: Apache-2.0
metadata:
  author: cutver
  version: "2.0"
---

## Activation Contract
Activate when the user asks to view, extract, preview, or render release notes, changelogs, or PR descriptions using Cutver.

## Hard Rules
- Never mutate `CHANGELOG.md` directly via ad-hoc string replacement or file writes; use `cutver bump` for automated updates.
- Use `cutver changelog latest` to extract notes for the current/most recent release.
- Use `cutver changelog show <version>` to extract historical release notes.
- Emit cleanly formatted markdown suitable for piping into GitHub Releases (`gh release create`) or release notifications.

## Decision Gates
1. **Target Selection**:
   - Latest release: use `latest`.
   - Historical release: use `show <version>` with explicit semver tag.
2. **Header Inclusions**:
   - Release body for GitHub/GitLab: default (omits outer version header).
   - Document view: pass `--include-header` to retain markdown H2 release title and date.
3. **Template Engine**:
   - Built-in format: default keep-a-changelog markdown rendering.
   - Custom template: pass `--template <path>` referencing a MiniJinja template.

## Execution Steps
1. Determine release version target (latest vs specific version).
2. For latest release notes:
   ```bash
   cutver changelog latest
   ```
   Or with header:
   ```bash
   cutver changelog latest --include-header
   ```
3. For historical release notes:
   ```bash
   cutver changelog show <version>
   ```
4. For custom template rendering:
   ```bash
   cutver changelog latest --template <template-path>
   ```
5. Direct output to terminal, pipe to file, or pass to downstream release tool:
   ```bash
   cutver changelog latest > RELEASE_NOTES.md
   ```

## Output Contract
- Stream extracted markdown release notes to standard output.
- On a terminal with colors on, headings, categories, and bullets are styled and markdown links become OSC 8 hyperlinks whose URLs are repeated in a footnote section.
- Colors are off when the stream is not a TTY, `NO_COLOR` is non-empty, or `CLICOLOR=0`; a `CLICOLOR_FORCE` value other than `0` forces them on. With colors off the markdown is printed byte-for-byte unstyled, so redirected output stays clean.
- Validate that extracted notes are non-empty and correspond to the requested release target.

## References
- Subcommands: `cutver changelog latest`, `cutver changelog show <version>`
- Template support: MiniJinja context, partial list — `version`, `previous_version`, `tag`, `date`, `is_prerelease`, `repo_owner`, `repo_name`, `compare_url`, the category blocks (`breaking`, `features`, `fixes`, `perf`, `refactor`, `docs`, `maintenance`, `other`, `all_changes`), `commits`, `contributors`.
- Full context of the installed build: `cutver changelog latest --json`. Template keys: `docs/reference/configuration.md` `[changelog]`. Filters and commit fields: `docs/templates/release-notes.md`.
