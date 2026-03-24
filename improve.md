End-of-session review to make the user more efficient in future sessions. Examines how the user interacted with Claude and proposes concrete improvements across skills, permissions, memory, and workflow.

## Goal

The goal is not just to fix bugs in skills — it's to reduce friction. Look for anything that slowed the user down, required extra back-and-forth, or needed manual intervention that could have been automated. Every approval prompt, correction, repeated instruction, or workaround is a candidate for elimination.

---

## Steps

### 1. Analyse the session

Read the full conversation and look for:

- **Friction points** — approval prompts, interrupted tool calls, commands the user had to explicitly allow, back-and-forth to clarify something that could have been inferred
- **Corrections** — explicit ("don't do X") or implicit (user re-stated a request, edited output, or pushed back)
- **Repeated instructions** — things the user had to say that are already in memory or skills but weren't followed
- **Manual steps** — things the user did themselves that Claude could have done
- **Skill gaps** — patterns that came up multiple times ad-hoc and would benefit from a reusable skill
- **Skill friction** — skills that required extra prompting, produced wrong output, or needed follow-up fixes

---

### 2. Produce a proposal

Group findings into these categories and present them all at once:

#### Skill updates
For each skill with actionable feedback: read the current file fresh from `/Users/davy/dev/claude-skills/` (do not rely on context — it may be stale). Draft the proposed change and explain what friction it removes.

#### New skills
For patterns that came up ad-hoc and would benefit from a reusable skill: propose a name, one-line description, and why it reduces friction based on this session.

#### Permission additions
For every approval prompt or interrupted tool call: assess whether it's safe to auto-approve globally (e.g. read-only `gh` commands, posting comments) vs. warranting case-by-case approval (destructive operations, pushes). Read `~/.claude/settings.json` and propose additions to `permissions.allow` using the `Bash(<prefix>:*)` pattern.

#### Memory updates
For any new patterns, preferences, or corrections that should persist: propose additions or updates to memory files. If something is now covered by a skill, trim the redundant memory entry.

---

### 3. Wait for approval

Present all proposals together. Wait for the user to approve or reject each one before making any changes.

---

### 4. Apply approved changes

- **Creating** new skills: use the `/new-skill <name>` skill (handles write, commit, and symlink)
- **Updating** existing skill files: edit directly with the Edit tool — do NOT use `/new-skill` (it creates an unnecessary symlink). Then commit all changes together below.
- Apply permission additions to `~/.claude/settings.json`
- Commit and push: `cd /Users/davy/dev/claude-skills && git add -A && git commit -m "Session improvements: <summary>" && git push`
- Update memory files and `MEMORY.md` index
