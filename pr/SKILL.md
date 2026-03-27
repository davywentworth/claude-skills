Full PR creation workflow: push branch, open PR, post review comment, wait for approval. Includes post-merge branch cleanup.

## Modes

- **`/pr`** (default) — create the PR, open in browser, post review comment, then stop
- **`/pr cleanup`** — post-merge branch deletion (run after PR is merged)

---

## Default mode (`/pr`)

1. **Push** the current branch: `git push -u origin HEAD`
2. **Create PR** using `mcp__github__create_pull_request`. Title format: `(#<issue-number>) <description>` — e.g. `(#2) Make CodeEditor resizable`. Derive the issue number from the branch name (e.g. `2-editor-resizable` → `#2`). Body follows the PR format from CLAUDE.md.
3. **Label the issue**: run `gh issue edit <issue-number> --add-label "in-review"` to mark it in-review.
4. **Open in browser**: run `open <PR_URL>`
5. **Post review comment**: Look for `/review` findings in the current conversation context. If found, use them. If not found, invoke the `/review` skill now to generate them. Then post the findings as a PR review comment using `mcp__github__create_pull_request_review` with `event: "COMMENT"`. Include all findings verbatim — even files that look good.
6. **Stop here.** Tell the user the PR is open and waiting for their review. Do NOT proceed or offer to do anything else — wait for the user to explicitly say the PR is merged or to run `/pr cleanup`.

---

## Cleanup mode (`/pr cleanup [branch-name]`)

Run this only after the user confirms the PR is merged.

Determine the branch name and worktree path:
- If run from inside a worktree: `branch=$(git branch --show-current)`, `worktree_path=$(pwd)`
- If a branch name is passed as an argument: use that; derive repo name via `basename $(git rev-parse --show-toplevel)` and worktree path as `../<repo>-<branch>`
- If neither: ask the user for the branch name

Then:
1. From the main repo (derive via `git worktree list | head -1 | awk '{print $1}'`): `git pull`
2. Remove the worktree: `git worktree remove <worktree_path>`
3. Delete branch locally: `git branch -d <branch-name>`
4. Delete branch remotely: `git push origin --delete <branch-name>`
5. Report what was deleted.
