---
name: review
description: This skill should be used when the user wants an antagonistic pre-commit code review on all staged and unstaged changes. Spawns parallel per-file review agents, fixes real issues automatically, and confirms lint passes.
user-invocable: true
effort: high
---

Run an antagonistic pre-commit code review on all staged and unstaged changes.

Steps:
1. Run `git diff HEAD` to get all uncommitted changes. If nothing, run `git diff main...HEAD` to review unpushed commits.
2. Identify every file that has changes.
3. For each changed file, spawn a separate Agent (subagent_type: code-reviewer) IN PARALLEL. Each agent receives:
   - The full diff for that file
   - The file's full current content (read it)
4. Collect all agents' findings. Print them in full to the user, grouped by file.
5. If any real issues were found (not just "looks good"), fix them — do not ask for confirmation first.
6. After fixing, run the project's linter and formatter in the affected package(s) to confirm clean. Detect from `package.json` scripts (npm run lint, npm run format:check), `go vet`, `ruff`, etc. If fixes introduce new issues, repeat from step 3 on the modified files.
7. Report what was fixed and what remains for the user to decide on.
