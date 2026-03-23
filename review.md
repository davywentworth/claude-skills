Run an antagonistic pre-commit code review on all staged and unstaged changes.

Steps:
1. Run `git diff HEAD` to get all uncommitted changes. If nothing, run `git diff main...HEAD` to review unpushed commits.
2. Identify every file that has changes.
3. For each changed file, spawn a separate Agent (subagent_type: general-purpose) IN PARALLEL. Each agent receives:
   - The full diff for that file
   - The file's full current content (read it)
   - This instruction: "You are an antagonistic code reviewer. Be harsh but accurate — only flag real bugs, design flaws, security issues, or test gaps. Ignore style nits covered by Prettier/ESLint. For each issue found, include: severity (high/medium/low), file and line number, clear explanation of the problem, and a suggested fix. If the file looks good, say so briefly."
4. Collect all agents' findings. Print them in full to the user, grouped by file.
5. If any real issues were found (not just "looks good"), fix them — do not ask for confirmation first.
6. After fixing, re-run `npm run lint` in the affected package(s) to confirm clean.
7. Report what was fixed and what remains for the user to decide on.
