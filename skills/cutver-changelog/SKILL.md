---
name: cutver-changelog
description: Extract release notes, format changelog entries, and render custom release templates for CI/CD, GitHub Releases, and PRs.
triggers:
  - changelog
  - cutver changelog
  - extract release notes
  - changelog latest
  - show release notes
---

# cutver-changelog

Guide for querying, extracting, and rendering release notes and changelogs using `cutver changelog`.

## Overview

`cutver changelog` allows developers and CI/CD pipelines to extract release notes for specific versions or the latest release directly from `CHANGELOG.md` or git history. This is particularly useful for:
- GitHub Actions workflows creating GitHub Releases (`gh release create`).
- Automated Slack / Discord release notifications.
- Pull request summaries and documentation generation.

---

## Extracting Release Notes for CI/CD & GitHub Releases

### Extract Latest Release Notes
To extract release notes for the most recent release:

```bash
# Output release notes without top-level header (ideal for GitHub Release body)
cutver changelog latest

# Include top-level version header (e.g. ## [1.2.0] - 2026-09-25)
cutver changelog latest --include-header
```

### GitHub Actions Workflow Example
Use the extracted notes directly in GitHub CLI commands:

```bash
# Extract latest notes into an environment variable or file
cutver changelog latest > RELEASE_NOTES.md

# Publish GitHub Release
gh release create "v$(cutver doctor | grep -oE '[0-9]+\.[0-9]+\.[0-9]+' | head -1)" \
  --title "Release v$(cutver doctor | grep -oE '[0-9]+\.[0-9]+\.[0-9]+' | head -1)" \
  --notes-file RELEASE_NOTES.md
```

### Extract Notes for a Specific Version
To retrieve historical changelog entries for any previous release:

```bash
# Extract notes for a specific version tag
cutver changelog show 1.2.0

# With header
cutver changelog show 1.2.0 --include-header
```

---

## MiniJinja Custom Templates

`cutver` supports custom MiniJinja (`jinja2`-compatible) templates for changelog rendering. Provide a custom template using the `--template` flag or configured in `cutver.toml`:

```bash
cutver changelog latest --template .github/templates/cutver/RELEASE.md
```

### Template Context Variables

When rendering templates, the following context variables are available:

| Variable | Type | Description | Example |
|:---|:---|:---|:---|
| `version` | `string` | The version number being released | `"1.3.0"` |
| `tag` | `string` | The full git tag name including prefix | `"v1.3.0"` |
| `previous_tag` | `string` | The previous git tag name (or empty if initial release) | `"v1.2.1"` |
| `date` | `string` | The release date in ISO format | `"2026-09-25"` |
| `breaking` | `string` | Pre-formatted markdown bullet points for breaking changes | `"- **api**: rewrite config parser"` |
| `features` | `string` | Pre-formatted markdown bullet points for `feat` commits | `"- **cli**: add init command"` |
| `fixes` | `string` | Pre-formatted markdown bullet points for `fix` commits | `"- **git**: trim newline from branch"` |
| `refactor` | `string` | Pre-formatted markdown bullet points for refactoring | `"- **runner**: deduplicate commands"` |
| `perf` | `string` | Pre-formatted markdown bullet points for performance improvements | `"- **cache**: optimize tool lookup"` |
| `contributors` | `list<string>` | Deduplicated commit authors / GitHub handles | `["@Row0902"]` |
| `compare_url` | `string` | Comparison URL between previous tag and current tag | `"https://github.com/cutver/cutver/compare/v0.4.0...v0.5.0"` |
| `commits` | `list<Commit>` | Raw list of commit objects (`hash`, `short_hash`, `subject`, `scope`, `author_name`, `body`) | `[{ subject: "...", ... }]` |

### Example Custom Template (`RELEASE.md`)

```jinja
**✨ What's Changed in {{ tag }}**
{% if breaking %}
### ⚠️ Breaking Changes
{{ breaking }}
{% endif -%}
{% if features %}
### 🚀 Features & Enhancements
{{ features }}
{% endif -%}
{% if fixes %}
### 🐛 Bug Fixes
{{ fixes }}
{% endif -%}
{% if contributors %}
### 👥 Contributors
{% for author in contributors -%}
- @{{ author }}
{% endfor -%}
{% endif -%}
{% if compare_url %}
---
**Full Diff**: {{ compare_url }}
{% endif -%}
```
