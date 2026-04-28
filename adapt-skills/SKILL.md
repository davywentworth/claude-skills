Browse a GitHub repo's Claude Code skills/commands, compare them to your existing workflow, and suggest which ones are worth adapting.

## Usage

- `/adapt-skills <github-repo-url>` — browse and adapt skills from an external repo
- `/adapt-skills update` — check all locally adapted skills for upstream changes

---

## Update mode (`/adapt-skills update`)

Check whether any locally adapted skills have drifted from their upstream source.

### 1. Find adapted skills

Scan all files in `/Users/davy/dev/claude-skills/` for the two-line credit block:

```
> Adapted from: <url>
> Upstream SHA: <sha>
```

Collect each skill's name, upstream file URL, and stored SHA.

### 2. Fetch current upstream SHAs

For each adapted skill, parse the URL to extract owner, repo, and file path. GitHub blob URLs follow the format `https://github.com/{owner}/{repo}/blob/{branch}/{path}` — extract accordingly. Then call `mcp__github__get_file_contents` to get the current SHA. Do all fetches in parallel.

### 3. Report status

Display a table:

```
────────────────┬──────────────────────────────────────────────────────┬────────────
 Skill          │ Upstream file                                        │ Status
────────────────┼──────────────────────────────────────────────────────┼────────────
 pr-respond     │ anutron/ai — skills/pr-respond/SKILL.md              │ Up to date
────────────────┼──────────────────────────────────────────────────────┼────────────
 devils-advocate│ anutron/ai — skills/devils-advocate/SKILL.md         │ Changed
────────────────┴──────────────────────────────────────────────────────┴────────────
```

### 4. For each changed skill

Fetch the upstream content and present a clear diff against the local version. For each difference, ask the user whether to adopt it.

If any changes are adopted:
- Apply them to the local SKILL.md
- Update the `> Upstream SHA:` line to the new SHA
- Commit the changes in `/Users/davy/dev/claude-skills/` with message: `Update <skill-name> from upstream (sha: <new-sha>)`

---

## Adapt mode (`/adapt-skills <github-repo-url>`)

### 1. Get the repo URL

If no URL was provided as an argument, ask: "What GitHub repo would you like to browse for skills?"

---

### 2. Discover skills in the repo

Use `WebFetch` or the GitHub MCP tools to find skills in the repo. Look in these locations (in order):

1. `.claude/commands/` — standard Claude Code commands directory
2. Any directory named `skills/`, `commands/`, or `prompts/`
3. README or docs that describe slash commands

Read every skill file you find.

---

### 3. Read your own skills

Read all files in `/Users/davy/dev/claude-skills/` and review the workflows described in memory (MEMORY.md). This gives you the baseline to compare against.

---

### 4. Analyze and compare

For each skill found in the external repo, evaluate:

- **Overlap**: does it duplicate something you already have?
- **Gap**: does it cover a workflow you don't have a skill for yet?
- **Quality**: is the skill well-written, clear, and opinionated?
- **Fit**: does it match the established workflows (feature branches, worktrees, PRs, issue-driven dev, etc.)?

---

### 5. Present recommendations

Output a ranked list of skills worth adapting. For each one:

- **Skill name** and what it does
- **Why it's useful** — what gap or improvement it addresses
- **Adaptation notes** — what would need to change to fit your workflow
- **Verdict**: `Adapt`, `Borrow ideas from`, or `Skip`

Ask the user: "Which of these would you like to bring in?"

---

### 6. Adapt chosen skills

For each skill the user wants to incorporate:

1. Use `mcp__github__get_file_contents` to fetch the file and capture both the content and its `sha` field.
2. Draft the adapted content. Include this credit block near the top (after any frontmatter):
   ```
   > Adapted from: <html_url of the upstream file>
   > Upstream SHA: <sha from the API response>
   ```
3. Adjust the content to match established conventions:
   - Branch naming, worktree paths, quality gates (`/review`, `/test-review`)
   - Issue-driven workflow, PR-first approach
   - Memory/MEMORY.md updates where relevant

---

### 7. Wire it up

Invoke the `/new-skill <name>` skill, passing the drafted content. `/new-skill` handles writing the file, the plannotator review, committing, pushing, and symlinking.

---

### 8. Update memory

Add each new skill to the Claude Skills section of the relevant memory file(s). If it covers something previously tracked as prose in memory, replace that prose with a skill reference.

---

Remind the user that new skills won't be available until the next session restart.
