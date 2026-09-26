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
| [`cutver-release`](./skills/cutver-release/SKILL.md) | `cutver-release` | Safe version bumping with mandatory dry-run simulation, conventional commit analysis, and changelog generation. | `release`, `cut release`, `bump version`, `semver bump`, `cutver bump` |
| [`cutver-doctor`](./skills/cutver-doctor/SKILL.md) | `cutver-doctor` | Workspace diagnostic checks detecting version drift across manifests and changelog desynchronization. | `doctor`, `cutver doctor`, `verify manifests`, `check changelog drift`, `release health check` |
| [`cutver-init`](./skills/cutver-init/SKILL.md) | `cutver-init` | Project onboarding and manifest autodiscovery for Rust, Node, Python, Gradle, and Tauri projects. | `init`, `cutver init`, `configure cutver`, `setup cutver`, `onboard cutver` |
| [`cutver-changelog`](./skills/cutver-changelog/SKILL.md) | `cutver-changelog` | Extract release notes for CI/CD, GitHub Releases, PR descriptions, and custom MiniJinja templates. | `changelog`, `cutver changelog`, `extract release notes`, `changelog latest`, `show release notes` |

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
