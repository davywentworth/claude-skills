---
name: fixit
description: Use when the user reports a bug that can be fixed without blocking current work. Creates a GitHub issue for tracking history, then backgrounds an agent in a worktree to fix and merge back.
---

> Adapted from: https://github.com/anutron/ai/blob/main/skills/fixit/SKILL.md
> Upstream SHA: 5b250a88df81f1e512672f86addddc0fe387be18

# Fixit

One-shot background bug fix. Creates a GitHub issue for tracking, spins up a worktree agent to fix it, merges back, and reports what changed.

## Arguments

- `$ARGUMENTS` — **Required.** Natural language description of the bug to fix. Optionally prefix with `#<issue-number>` to link an existing issue instead of creating one.

If no arguments provided, reply: `Usage: /fixit <describe the bug>` and stop.

## Context

- Current branch: !`git branch --show-current`
- Project root: !`pwd`
- Main repo root: !`git worktree list --porcelain 2>/dev/null | head -1 | sed 's/^worktree //'`
- Repo: !`git remote get-url origin 2>/dev/null | sed 's/.*github.com[:/]//' | sed 's/\.git$//'`

---

## Instructions

### 1. Parse Issue Reference

If `$ARGUMENTS` starts with `#<number>` (e.g. `#42 the login button is broken`):
- Extract the issue number as the tracking issue
- Use the remaining text as the bug description
- Skip Step 2

Otherwise proceed to Step 2.

### 2. Create GitHub Issue

Use `mcp__github__create_issue` to file a tracking issue before dispatching the fix. Parse owner and repo from the `Repo` context above (format: `owner/repo`).

- **title**: short imperative description (e.g. "Fix login button unresponsive on mobile")
- **body**:
  ```
  ## Bug Report

  <user's full bug description>

  ---
  *Being fixed automatically via `/fixit`.*
  ```
- **labels**: `["bug"]` — omit if label doesn't exist (don't fail)

Save the returned issue number for the branch name and agent prompt.

### 3. Triage (main thread only — ≤30 seconds)

You are a dispatcher, not a debugger. Do NOT read source files or investigate root causes.

- Run up to 3 `Glob`/`Grep` calls (file paths only, no content reads) to locate likely files
- If the description is ambiguous, echo back a 1-line interpretation and proceed — don't block on clarification

### 4. Create Worktree

Resolve the main repo root first — fixit may be invoked from inside a worktree:

```bash
MAIN_REPO=$(git worktree list --porcelain 2>/dev/null | head -1 | sed 's/^worktree //')
SLUG="fixit-<issue-number>-<short-slug>"
git worktree add -b "$SLUG" "$MAIN_REPO/.claude/worktrees/$SLUG" HEAD
```

If the branch already exists, clean up first:
```bash
git worktree remove "$MAIN_REPO/.claude/worktrees/$SLUG" --force 2>/dev/null
git branch -D "$SLUG" 2>/dev/null
```

### 5. Dispatch Background Agent

Use the `Agent` tool with `run_in_background: true`:

```
## Bug Fix: <title> (GitHub issue #<number>)

### Context
- Main repo root: <$MAIN_REPO>
- Working directory: <worktree path>
- Branch: <SLUG>
- Tracking issue: #<issue-number>

### Bug Description
<user's description>

### Files Likely Involved
<from triage, or "Explore the codebase to find the relevant code">

### Instructions
1. Read CLAUDE.md for project conventions and architecture
2. Explore the codebase to understand and reproduce the problem
3. Implement the fix — keep it minimal, don't refactor surrounding code
4. Run tests for the affected package(s):
   - Backend: cd backend && npm test -- --run
   - Frontend: cd frontend && npm test -- --run
5. Run lint: npm run lint in affected package(s)
6. Commit with message: "Fix #<issue-number>: <short description>"
   The "Fix #N" prefix auto-closes the GitHub issue when merged to main.
7. Report status: DONE | DONE_WITH_CONCERNS | BLOCKED

### Code Style (from CLAUDE.md)
- No semicolons, single quotes, 2-space indent
- Trailing commas where valid in ES5
- Max line width: 100 characters

### Constraints
- Work ONLY in your worktree directory: <worktree path>
- Follow existing patterns — do not introduce new abstractions
- If tests fail after your fix, investigate and resolve before reporting DONE
```

### 6. Confirm to User

Print one line and return control immediately:

```
Fixit dispatched — agent working on "<short title>" in background. Tracking: #<issue-number>
```

---

## On Agent Completion

When the background agent reports back:

### Success Path

```bash
git checkout <original-branch>
git merge <SLUG> --no-edit
```

**If merge succeeds:**
```bash
git worktree remove "$MAIN_REPO/.claude/worktrees/$SLUG" --force
git branch -D "$SLUG"
```

Report to user:
```
✅ Fixit merged: <short title>
  <1-2 line summary of what changed>
  Issue #<number> will auto-close when pushed to main.
```

**If merge conflicts:**
```bash
git merge --abort
```

Report:
```
⚠️ Fixit conflict: <short title>
  Worktree preserved at $MAIN_REPO/.claude/worktrees/<SLUG> for manual resolution.
  Issue #<number> remains open.
```

### Failure Path

Clean up the worktree. Leave the GitHub issue open for manual follow-up. Report:

```
❌ Fixit failed: <short title>
  <brief reason from agent>
  Issue #<number> left open for manual follow-up.
```

---

## Rules

- **Never read source code in the main thread** — agents do that
- **Always create a GitHub issue** unless `#<number>` was provided
- **Never block the user** — dispatch and return immediately
- **One bug, one agent, one worktree**
- **Triage search budget**: max 3 Glob/Grep calls, zero file reads
