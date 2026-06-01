---
name: implement
description: This skill should be used when the user wants to implement a GitHub issue end-to-end. Validates readiness, creates a git worktree, implements the plan, runs quality gates, opens a PR, and links the issue. Invoke as /implement <issue-number>.
user-invocable: true
effort: high
---

Implement a GitHub issue end-to-end: validate it's ready, create a git worktree, implement the plan, run quality gates, open a PR, and link the issue.

## Usage

`/implement <issue-number>`

---

## Steps

### 1. Fetch & validate

1. Fetch issue details using `mcp__github__get_issue`, then fetch comments separately: `gh api repos/{owner}/{repo}/issues/{number}/comments` (MCP has no get-comments tool). Derive owner/repo from `gh repo view --json owner,name`.
2. If the issue has the `needs-detail` label → **stop**. Tell the user to run `/issue plan <number>` first.
3. If no comment contains any of `## Steps`, `## Implementation`, `## Plan`, or `## Changes` → **stop**. Tell the user to run `/issue plan <number>` first.

---

### 2. PR splitting check

Default to a **single PR** and proceed to step 3. If the issue plan has 3 or more clearly independent stages, note the option to split in a single line but do not block — proceed with a single PR unless the user explicitly requested a split.

If a split is requested (in the invocation or issue plan): read the plan and determine logical split points (e.g. DB layer → route layer → frontend). Implement each chunk as a separate PR in sequence — complete all steps through step 6 for each chunk before starting the next. Use `Part of #<number>` in each PR body except the last, which gets `Closes #<number>`.

---

### 3. Create worktree

Branch name convention: `<number>-<kebab-case-title>` (e.g. issue #42 "Add dark mode" → `42-add-dark-mode`). For split PRs, append a suffix: `42-add-dark-mode-db`, `42-add-dark-mode-api`, etc.

Derive the repo name dynamically: `repo=$(basename $(git rev-parse --show-toplevel))`. Create a git worktree from the main repo root:
```
git worktree add ../<repo>-<branch> -b <branch>
```

Immediately after creating the worktree, install dependencies based on the project type. Detect from `package.json`, `pyproject.toml`, `go.mod`, or `Cargo.toml` at the repo root:
- **Node/JS**: run `npm install` (or `yarn install` / `pnpm install`) in each package directory. Worktrees do not inherit `node_modules` — tests will error if skipped.
- **Python**: run `pip install -e .` or `pip install -r requirements.txt` if a requirements file exists.
- **Go / Rust**: no install step needed — dependencies resolve at build time.

All subsequent commands (implementation, tests, commits, quality gates) run from the worktree directory: `../<repo>-<branch>`.

---

### 4. Implement

Follow the plan from the issue comments. Write or update tests alongside the implementation — do not treat them as a separate phase.

---

### 5. Quality gates

1. Invoke the `/test-review` skill on all new or changed test files.
2. Invoke the `/review` skill on all changes.
3. Commit once both pass.

---

### 6. Hand off

Invoke the `/pr` skill directly to push the branch, open the PR, and post the review comment. `/pr` will stop after opening — the user reviews and merges manually. Remind them to include `Closes #<number>` (or `Part of #<number>` for non-final chunks in a split) in the PR body.
