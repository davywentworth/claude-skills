Full PR creation workflow: push branch, open PR, post review comment, wait for approval. Includes post-merge branch cleanup.

## Modes

- **`/pr`** (default) — create the PR, open in browser, post review comment, then stop
- **`/pr cleanup`** — post-merge branch deletion (run after PR is merged)

---

## Default mode (`/pr`)

1. **Push** the current branch: `git push -u origin HEAD`
2. **Create PR** using `gh pr create` with a title and body summarizing the changes (follow the PR format from CLAUDE.md)
3. **Open in browser**: run `open <PR_URL>`
4. **Post review comment**: If `/review` was run earlier in this session, post a PR review comment with the findings using `gh pr review <PR_URL> --comment --body "..."`. Include all findings verbatim — even files that look good. If `/review` has NOT been run this session, skip this step and note it to the user.
5. **Stop here.** Tell the user the PR is open and waiting for their review. Do NOT proceed or offer to do anything else — wait for the user to explicitly say the PR is merged or to run `/pr cleanup`.

---

## Cleanup mode (`/pr cleanup [branch-name]`)

Run this only after the user confirms the PR is merged.

Determine the branch name and worktree path:
- If run from inside a worktree: `branch=$(git branch --show-current)`, `worktree_path=$(pwd)`
- If a branch name is passed as an argument: use that; derive worktree path as `../tech-bridge-<branch>`
- If neither: ask the user for the branch name

Then:
1. From the main repo (`/Users/davy/dev/tech-bridge`): `git pull`
2. Remove the worktree: `git worktree remove <worktree_path>`
3. Delete branch locally: `git branch -d <branch-name>`
4. Delete branch remotely: `git push origin --delete <branch-name>`
5. Report what was deleted.
