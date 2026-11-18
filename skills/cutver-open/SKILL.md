---
name: cutver-open
description: "Trigger: cutver open, open release notes, open release tag, compare tags, open cutver repo. Open the repository remote, a release tag, or a tag comparison with cutver open."
license: Apache-2.0
metadata:
  author: cutver
  version: "2.0"
---

## Activation Contract
Activate when the user asks to open the repository page, a release tag, or a tag comparison in a browser via `cutver open`.

## Hard Rules
- `cutver open [TARGET]` builds one URL from the git remote of the repository root and hands it to a browser command; no documentation or changelog URL is resolved.
- The remote is read with `git config --get remote.origin.url` and normalized before use: `git@host:owner/repo.git` and `ssh://git@host/owner/repo` become `https://host/owner/repo`, and a trailing `.git` is stripped.
- `TARGET` is parsed as follows:
  - omitted or empty: `Repo`, the normalized remote URL.
  - `compare` (case-insensitive): comparison view headed at `HEAD`.
  - `<base>...<head>` (three dots): comparison between two tags.
  - anything else: a version, templated through `[git] tag_prefix` into `<remote>/releases/tag/<tag>`.
  - `docs`, `changelog`, and `release` are not targets: each is read as a version and yields a release-tag URL such as `.../releases/tag/vdocs`.
- Pass `--print-url` (visible alias `--dry-run`) to print the URL instead of launching a browser.
- Pass `--browser <PROGRAM>` to override the launcher; the value is used as the program name and receives the URL as its only argument.

## Decision Gates
1. **Target Selection**:
   - Repository page: omit the argument.
   - Release page: the version, e.g. `1.2.3` under the default prefix becomes `.../releases/tag/v1.2.3`.
   - Comparison: `compare`, or `v1.0.0...v1.1.0`.
2. **Environment Capability**:
   - Interactive desktop: the URL goes to the platform launcher (`open`, `cmd /c start`, `xdg-open`, or `wslview` when WSL provides it). A non-empty `BROWSER` variable takes precedence over the platform default.
   - Headless / SSH / Container: run with `--print-url` and read stdout; `open` prints no fallback URL on its own and exits `1` when the launcher fails.

## Execution Steps
1. Pick the target: none, a version, `compare`, or `<base>...<head>`.
2. Resolve the URL first and read it from stdout:
   ```bash
   cutver open --print-url
   cutver open 1.2.3 --print-url
   ```
3. Launch it only when the user asked for navigation:
   ```bash
   cutver open 1.2.3
   ```
4. Report what `--print-url` printed, or the launcher outcome. Exit `1` means the URL could not be resolved (no git `origin` remote, or `--config` failed to load) or the browser command failed; the error message names which.

## Output Contract
- With `--print-url`: stdout is the resolved URL alone and the exit code is `0`.
- Without it: exit `0` after the browser command exits successfully, `1` with a message on stderr otherwise.

## References
- Subcommand: `cutver open [TARGET] [--print-url] [--browser <BROWSER>]`; `cutver changelog open` takes the same arguments.
- URL shapes on the normalized remote `<remote>`: `<remote>`; `<remote>/releases/tag/<tag>`; `<remote>/compare/HEAD` for a bare `compare` and `<remote>/compare/<a>...<b>` for a range. A GitLab-shaped remote (the URL contains `gitlab.com` or `/-/`) uses `/-/releases/<tag>`, `/-/compare?to=HEAD`, and `/-/compare/<a>...<b>`.
- Tag formatting: `[git] tag_prefix` (default `"v"`).
