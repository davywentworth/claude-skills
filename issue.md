GitHub issue workflow skill. Handles brain dumps, listing, and planning.

Detect the current repo with `gh repo view --json nameWithOwner -q .nameWithOwner`.

## Usage modes

Determine mode from how the skill was invoked:

- **`/issue list`** — list open issues
- **`/issue plan <number>`** — create a plan for an issue
- **Default (no args, or one or more lines starting with "Issue:")** — brain dump mode

---

## Brain dump mode

The user may provide one or more items in the form `Issue: <description>`, or invoke `/issue` with a description directly.

1. Fetch all open issues from the repo.
2. For each item, check for duplicates by comparing against existing issue titles and bodies. If a duplicate exists, tell the user and skip it.
3. For each non-duplicate, create a GitHub issue with the `needs-detail` label.
4. Report what was created and what was skipped as duplicates.

---

## List mode (`/issue list`)

1. Fetch all open issues and open PRs from the repo in parallel.
2. Output a solid full-width line using box-drawing characters (`─` U+2500, repeated ~80 times), then display open PRs (if any), then open issues. For each issue, clearly indicate if it has the `needs-detail` label.

---

## Plan mode (`/issue plan <number>`)

Follow these steps in order — do not skip ahead:

1. Fetch the issue details (title, body, comments) using `gh issue view <number>`.
2. If the issue body is empty (no description), ask the user for context and detail before proceeding. Incorporate their response into the plan.
3. Write a thorough implementation plan to `plans/<issue-slug>.md` where `<issue-slug>` is the issue number + kebab-case title (e.g. `plans/42-add-dark-mode.md`).
4. Open the plan in plannotator for review using the `plannotator-annotate` skill. Remind the user that they must interact with at least one element in the UI (e.g. a 👍 on the title) before closing — just closing the tab will hang the process.
5. After the user approves the plan, post it as a comment on the GitHub issue using `gh issue comment`, then remove the `needs-detail` label with `gh issue edit <number> --remove-label needs-detail`.
6. **Stop here.** Wait for the user's explicit go-ahead before writing any code. When they give it, tell them to run `/implement <number>` to begin implementation.
