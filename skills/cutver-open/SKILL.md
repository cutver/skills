---
name: cutver-open
description: "Trigger: cutver open, open changelog, open release notes, open cutver docs, open cutver repo. Open Cutver documentation, release notes, changelogs, or repository in browser or terminal."
license: Apache-2.0
metadata:
  author: cutver
  version: "2.0"
---

## Activation Contract
Activate when the user asks to open, view, or navigate to Cutver documentation, release notes, repository links, or local changelogs via `cutver open`.

## Hard Rules
- Use `cutver open <target>` for terminal and browser navigation.
- Valid targets are strictly: `release`, `docs`, `repo`, `changelog`.
- Support OSC 8 hyperlinks in modern terminal emulators.
- Handle headless, SSH, and WSL environments gracefully by printing URL fallback to stdout when a graphical browser cannot be launched.

## Decision Gates
1. **Target Selection**:
   - `docs`: Official Cutver documentation and guides.
   - `repo`: Cutver GitHub source repository.
   - `release`: Latest GitHub release page or current tag release.
   - `changelog`: Project changelog file or rendered changelog view.
2. **Environment Capability**:
   - Interactive desktop: open in system default browser.
   - Headless / SSH / Container: output resolved URL directly to stdout.

## Execution Steps
1. Identify desired target (`docs`, `repo`, `release`, `changelog`).
2. Execute command:
   ```bash
   cutver open <target>
   ```
3. If running in a headless or container environment, pass `--print-url` (or capture stdout URL fallback):
   ```bash
   cutver open <target> --print-url
   ```
4. Confirm user was navigated to target or display the printed URL.

## Output Contract
- Indicate the opened target URL or destination.
- In terminal-only environments, display clickable OSC 8 hyperlink or full HTTPS URL.

## References
- Subcommand: `cutver open [docs|repo|release|changelog]`
- Documentation URL: `https://github.com/cutver/cutver#readme`
