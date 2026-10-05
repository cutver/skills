# Cutver Agent Skills Catalog

Official agent skills for [Cutver](https://github.com/cutver/cutver) — the blazing-fast, polyglot release automation and semantic version management tool.

These skills enable AI coding assistants (Pi, Claude Code, Cursor, Copilot, Codex, and others) to safely inspect repository health, onboard multi-manifest workspaces, simulate releases, and publish changelogs following best practices.

---

## Installation

Install all Cutver skills into your current workspace using your preferred package runner:

### npm
```bash
npx skills add cutver/skills
```

### bun
```bash
bunx skills add cutver/skills
```

### pnpm
```bash
pnpm dlx skills add cutver/skills
```

You can also install specific individual skills:
```bash
npx skills add cutver/skills --skill cutver-release
```

---

## Skills Catalog

| Skill | Name | Description | Common Triggers |
|:---|:---|:---|:---|
| [`cutver-init`](./skills/cutver-init/SKILL.md) | `cutver-init` | Manifest autodiscovery and `cutver.toml` generation across polyglot workspaces. | `cutver init`, `init cutver`, `onboard cutver`, `scaffold cutver`, `configure cutver` |
| [`cutver-doctor`](./skills/cutver-doctor/SKILL.md) | `cutver-doctor` | Workspace diagnostic checks detecting version drift across manifests and changelog desynchronization. | `cutver doctor`, `doctor`, `check changelog drift`, `verify manifests`, `release health check` |
| [`cutver-release`](./skills/cutver-release/SKILL.md) | `cutver-release` | Safe version bumping with mandatory dry-run simulation, conventional commit analysis, and changelog generation. | `cutver bump`, `cutver release`, `cut release`, `bump version`, `semver bump`, `first release` |
| [`cutver-changelog`](./skills/cutver-changelog/SKILL.md) | `cutver-changelog` | Release note extraction for CI/CD, GitHub Releases, PR descriptions, and custom MiniJinja templates. | `cutver changelog`, `changelog latest`, `cutver changelog show`, `extract release notes`, `changelog template` |
| [`cutver-actions`](./skills/cutver-actions/SKILL.md) | `cutver-actions` | GitHub Actions workflow scaffolding with `cutver/setup` and `cutver/release` orchestration. | `cutver action`, `cutver/setup`, `cutver/release`, `github actions cutver`, `ci release workflow` |
| [`cutver-containers`](./skills/cutver-containers/SKILL.md) | `cutver-containers` | Containerized execution with Docker, Podman, and WSL Containers via `ghcr.io/cutver/cutver`. | `cutver container`, `cutver docker`, `cutver podman`, `cutver wslc`, `ghcr.io/cutver/cutver` |
| [`cutver-install`](./skills/cutver-install/SKILL.md) | `cutver-install` | Platform-specific CLI installation via Homebrew, Scoop, Cargo, or verified binary releases. | `install cutver`, `setup cutver cli`, `brew install cutver`, `scoop install cutver`, `download cutver` |
| [`cutver-open`](./skills/cutver-open/SKILL.md) | `cutver-open` | Fast terminal and browser navigation to documentation, changelogs, releases, and repository. | `cutver open`, `open changelog`, `open release notes`, `open cutver docs`, `open cutver repo` |

---

## Golden Rule for Agents

> **Mandatory Dry Run**: Agents using the `cutver-release` skill **must always** execute `cutver bump auto --dry-run` first to preview the calculated version bump, affected manifests, and commit history before performing any mutating operations or git commits.

---

## Agent Usage Instructions

### Pi Coding Agent
Pi automatically detects skills declared in the workspace `.skills` directory or global locations.
```bash
# In Pi, simply ask:
"Run a health check on my project manifests with cutver"
"Simulate the next release bump using cutver"
```

### Claude Code
Claude Code reads skills defined under the Open Agent Skills standard.
```bash
claude
# Inside Claude Code:
"Onboard this repository to Cutver"
"Check if any manifest versions have drifted"
```

### Cursor
Add this repository or installed skills into `.cursorrules` or prompt directly in Cursor Chat:
```markdown
Follow the Cutver release procedure in skills/cutver-release/SKILL.md.
Always run `cutver bump auto --dry-run` before modifying manifests.
```

### GitHub Copilot
Prompt GitHub Copilot in your IDE or CLI:
```text
@workspace /cutver-doctor Run health checks on our Cargo.toml and package.json manifests.
```

---

## License

MIT © [Cutver Contributors](https://github.com/cutver/cutver)
