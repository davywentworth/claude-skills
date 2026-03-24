Implement a GitHub issue end-to-end: validate it's ready, create a git worktree, implement the plan, run quality gates, open a PR, and link the issue.

## Usage

`/implement <issue-number>`

---

## Steps

### 1. Fetch & validate

1. Run `gh issue view <number> --json title,body,labels,comments`
2. If the issue has the `needs-detail` label → **stop**. Tell the user to run `/issue plan <number>` first.
3. If no comment contains `## Steps`, `## Implementation`, or `## Plan` → **stop**. Tell the user to run `/issue plan <number>` first.

---

### 2. PR splitting check

Ask: "Should this be implemented as a single PR, or split into smaller ones?"

Wait for the user's answer before continuing.

- **Single PR**: proceed to step 3.
- **Split**: read the plan and determine logical split points (e.g. DB layer → route layer → frontend). Present the proposed breakdown and wait for the user to confirm it before starting. Implement each chunk as a separate PR in sequence — complete all steps through step 6 for each chunk before starting the next. Use `Part of #<number>` in each PR body except the last, which gets `Closes #<number>`.

---

### 3. Create worktree

Branch name convention: `<number>-<kebab-case-title>` (e.g. issue #42 "Add dark mode" → `42-add-dark-mode`). For split PRs, append a suffix: `42-add-dark-mode-db`, `42-add-dark-mode-api`, etc.

Create a git worktree from the main repo root:
```
git worktree add ../tech-bridge-<branch> -b <branch>
```

All subsequent commands (implementation, tests, commits, quality gates) run from the worktree directory: `../tech-bridge-<branch>`.

---

### 4. Implement

Follow the plan from the issue comments. Write or update tests alongside the implementation — do not treat them as a separate phase.

---

### 5. Quality gates

1. Run `/test-review` on all new or changed test files.
2. Run `/review` on all changes.
3. Commit once both pass.

---

### 6. Open PR & link issue

1. Follow the `/pr` workflow — push branch, create PR, open in browser, post review comment. Ensure the PR body includes `Closes #<number>` (or `Part of #<number>` for non-final chunks in a split).
2. Post a comment on the issue linking the branch and PR:
   ```
   gh issue comment <number> --body "Implementation started on branch `<branch>`: <PR_URL>"
   ```
