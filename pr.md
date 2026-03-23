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

## Cleanup mode (`/pr cleanup`)

Run this only after the user confirms the PR is merged.

1. Get the current branch name: `git branch --show-current`
2. Switch to main and pull: `git checkout main && git pull`
3. Delete branch locally: `git branch -d <branch-name>`
4. Delete branch remotely: `git push origin --delete <branch-name>`
5. Report what was deleted.
