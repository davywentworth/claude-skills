GitHub issue workflow skill. Handles brain dumps, listing, and planning.

Use GitHub MCP tools (`mcp__github__*`) for all GitHub operations — do NOT use `gh` CLI via Bash unless no MCP tool exists for the operation. The repo owner/name are parameters to MCP calls; ask the user if ambiguous.

## Usage modes

Determine mode from how the skill was invoked:

- **`/issue list`** — list open issues
- **`/issue plan <number>`** — create a plan for an issue
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
2. Output a solid full-width line using box-drawing characters (`─` U+2500, repeated ~80 times), then display open PRs (if any), then open issues. For each issue, clearly indicate if it has the `needs-detail` label.

---

## Plan mode (`/issue plan <number>`)

Follow these steps in order — do not skip ahead:

1. Fetch the issue details (title, body, comments) using `mcp__github__get_issue`.
2. If the issue body is empty (no description), ask the user for context and detail before proceeding. Incorporate their response into the plan.
3. Write a thorough implementation plan to `plans/<issue-slug>.md` where `<issue-slug>` is the issue number + kebab-case title (e.g. `plans/42-add-dark-mode.md`).
4. Invoke the `/devils-advocate` skill on the plan. Present the full analysis to the user. Revise the plan to address any top concerns before proceeding.
5. Open the revised plan in plannotator for review using the `plannotator-annotate` skill. Remind the user that they must interact with at least one element in the UI (e.g. a 👍 on the title) before closing — just closing the tab will hang the process.
6. After the user approves the plan, post it as a comment on the GitHub issue using `mcp__github__add_issue_comment`, then remove the `needs-detail` label using `gh issue edit <number> --remove-label needs-detail` (no MCP tool available for label removal).
7. **Stop here.** Wait for the user's explicit go-ahead before writing any code. When they give it, tell them to run `/implement <number>` to begin implementation.
