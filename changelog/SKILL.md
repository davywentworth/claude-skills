---
name: changelog
description: Use when the user asks for a changelog, release notes, or summary of recent changes. Generates a human-readable changelog from git history.
allowed-tools: Bash(git log:*), Bash(git diff:*), Bash(git show:*), Bash(git tag:*), Bash(pbcopy:*)
---

> Adapted from: https://github.com/anutron/ai/blob/main/skills/changelog/SKILL.md
> Upstream SHA: 0d101202a97f3decb9656205614098d960d05401

## Context

- Current branch: !`git branch --show-current`
- Latest tag: !`git describe --tags --abbrev=0 2>/dev/null | head -1`
- Recent commits (last 7 days): !`git log --oneline --since="7 days ago" 2>/dev/null | head -20`

## Arguments

- `$ARGUMENTS` - Optional: time period (e.g. "2 weeks", "since v1.2.0", "last 30 days", "2025-01-01..2025-02-01")

## Your task

Generate a well-organized, human-readable changelog from recent git history.

### Step 1: Determine the time range

If the user provided a time period in `$ARGUMENTS`, use it. Otherwise ask what period to cover. Suggest useful options based on context above:

- Since a specific tag (if tags exist)
- Last N days/weeks
- A date range
- Since a specific commit

### Step 2: Gather commits

Run `git log` for the chosen time range with full commit messages:

```bash
git log --format="%H%n%s%n%b%n---END---" <range>
```

If there are few commits, also inspect diffs to understand scope:

```bash
git diff --stat <range>
```

### Step 3: Analyze and categorize

Read through all commit messages and diffs. Group changes into categories — use only those with entries:

- **Added** — new features, commands, tools, or capabilities
- **Changed** — modifications to existing behavior, UI updates, refactors
- **Fixed** — bug fixes
- **Removed** — deleted features, deprecated code removal
- **Infrastructure** — CI/CD, build system, dependency updates, tooling
- **Documentation** — README, docs, comments

### Step 4: Write the changelog

Write a changelog that is:

- **Grouped by category** with clear headers
- **Concise but meaningful** — each entry explains *what* and *why*, not just the commit message verbatim
- **Deduplicated** — combine related commits into a single entry
- **Ordered by significance** within each category
- **Free of noise** — omit trivial changes like typo fixes or whitespace unless nothing else happened

### Step 5: Present and offer to copy

Print the changelog in this format:

```
## Changelog: <description of range>

### Added
- Entry here

### Changed
- Entry here

### Fixed
- Entry here
```

Then offer to:
1. Copy to clipboard (`pbcopy`)
2. Save to a `CHANGELOG.md` file
