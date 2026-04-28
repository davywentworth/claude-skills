GitHub issue workflow skill. Handles brain dumps, listing, and planning.

Use GitHub MCP tools (`mcp__github__*`) for all GitHub operations — do NOT use `gh` CLI via Bash unless no MCP tool exists for the operation. The repo owner/name are parameters to MCP calls; ask the user if ambiguous.

## Usage modes

Determine mode from how the skill was invoked:

- **`/issue list`** — list open issues
- **`/issue plan <number>`** — create a plan for an issue
- **`/issue evaluate <number>`** — review and revise an existing plan already posted as an issue comment
- **Default with description (one or more lines starting with "Issue:")** — brain dump mode
- **Default with no args and no description** — list mode (same as `/issue list`)

---

## Brain dump mode

The user may provide one or more items in the form `Issue: <description>`, or invoke `/issue` with a description directly.

1. Fetch all open issues from the repo using `mcp__github__list_issues`.
2. For each item, check for duplicates by comparing against existing issue titles and bodies. If a duplicate exists, tell the user and skip it.
3. For each non-duplicate, create a GitHub issue with the `needs-detail` label using `mcp__github__create_issue`.
4. Report what was created and what was skipped as duplicates.

---

## List mode (`/issue list`)

1. Fetch all open issues and open PRs from the repo in parallel using `mcp__github__list_issues` and `mcp__github__list_pull_requests`.
2. Display results as a box-drawing table with columns: `#`, `Title`, `Labels`. Use `─`, `│`, `┬`, `┼`, `┴`, `├`, `┤` characters. Separate every row with a full `────┼────` divider line. Show PRs first (if any), then issues. End with a summary line (e.g. `7 issues · 2 PRs`). Example format:

```
────────────┬──────────────────────────────────────────────────────────────┬───────────────
 #          │ Title                                                        │ Labels
────────────┼──────────────────────────────────────────────────────────────┼───────────────
 #20        │ Extend curriculum support beyond TypeScript, JS, and Python  │ needs-detail
────────────┼──────────────────────────────────────────────────────────────┼───────────────
 #2         │ Editor needs to be resizable                                 │
────────────┴──────────────────────────────────────────────────────────────┴───────────────
 2 issues · 0 PRs
```

---

## Evaluate mode (`/issue evaluate <number>`)

For when a plan already exists as an issue comment and needs review/revision.

1. Fetch the issue details using `mcp__github__get_issue`, then fetch comments separately: `gh api repos/{owner}/{repo}/issues/{number}/comments` (MCP has no get-comments tool).
2. Identify the most recent plan comment.
3. Invoke the `/devils-advocate` skill on the plan. Present the full analysis to the user.
4. Incorporate any accepted concerns into a revised plan.
5. Post the revised plan as a new comment on the issue using `mcp__github__add_issue_comment`, noting it supersedes the previous plan.

---

## Plan mode (`/issue plan <number>`)

Follow these steps in order — do not skip ahead:

1. Fetch the issue details using `mcp__github__get_issue`, then fetch comments separately: `gh api repos/{owner}/{repo}/issues/{number}/comments` (MCP has no get-comments tool). Use the body + comments together as context.
2. If the issue body is empty (no description), ask the user for context and detail before proceeding. Incorporate their response into the plan.
3. Spawn an Explore agent (subagent_type: Explore) to gather codebase context relevant to the issue. Give it the issue title and description, and instruct it to: find files likely to be touched, identify existing patterns to follow (naming, structure, conventions), and surface any related code that the plan should account for. Incorporate its findings into the plan.
4. Write a thorough implementation plan to `plans/<issue-slug>.md` where `<issue-slug>` is the issue number + kebab-case title (e.g. `plans/42-add-dark-mode.md`).
5. Invoke the `/devils-advocate` skill on the plan. Present the full analysis to the user. Revise the plan to address any top concerns before proceeding.
6. Open the revised plan in plannotator for review using the `plannotator-annotate` skill. Remind the user that they must interact with at least one element in the UI (e.g. a 👍 on the title) before closing — just closing the tab will hang the process.
7. After the user approves the plan, post it as a comment on the GitHub issue using `mcp__github__add_issue_comment`, then remove the `needs-detail` label using `gh issue edit <number> --remove-label needs-detail` (no MCP tool available for label removal).
8. **Stop here.** Wait for the user's explicit go-ahead before writing any code. When they give it, tell them to run `/implement <number>` to begin implementation.
