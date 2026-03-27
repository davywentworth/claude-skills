End-of-session review to make the user more efficient in future sessions. Examines how the user interacted with Claude and proposes concrete improvements across skills, permissions, memory, and workflow.

## Goal

Make interactions simpler without sacrificing power. The user should be able to say less and get more — shorter inputs, fewer approvals, less back-and-forth — while Claude handles the same quality and depth of work. Look for anything that slowed the user down, required extra clarification, needed manual intervention, or could have been inferred. Every approval prompt, correction, repeated instruction, or workaround is a candidate for elimination.

---

## Steps

### 1. Analyze the session

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

#### 2a. Skill updates
For each skill with actionable feedback: read the current file fresh from `/Users/davy/dev/claude-skills/` (do not rely on context — it may be stale). Draft the proposed change and explain what friction it removes. Also check all skills for:
- **Hardcoded project-specific values**: absolute paths, repo names, usernames, port numbers, or org names — flag any that should be derived dynamically (e.g. `git rev-parse`, `gh repo view`, `pwd`) or parameterised.
- **Missing skill delegation**: any skill that makes code changes and commits without first invoking `/review` is missing a quality gate. Flag it and propose adding the invocation before the commit step.

#### 2b. New skills
For patterns that came up ad-hoc and would benefit from a reusable skill: propose a name, one-line description, and why it reduces friction based on this session.

#### 2c. Permission additions
Scan back through the full conversation and explicitly enumerate every tool call made this session. For each one, note whether it ran without prompting or triggered an approval prompt. Do not summarise — list them. Only after completing this audit conclude whether there are permission gaps. For each prompted call, assess whether it's safe to auto-approve globally (e.g. read-only `gh` commands, posting comments) vs. warranting case-by-case approval (destructive operations, pushes). Read `~/.claude/settings.json` and propose additions to `permissions.allow`. Syntax differs by tool:
- **`Bash`**: prefix syntax — `Bash(cd /some/path && git:*)` — matches commands starting with that string
- **`Edit`**: glob syntax — `Edit(/some/path/**)` — matches file paths using glob patterns

Prefer directory-scoped rules over broad ones when the operation is only safe in a specific location.

#### 2d. Memory updates
For any new patterns, preferences, or corrections that should persist: propose additions or updates to memory files. If something is now covered by a skill, trim the redundant memory entry.

When reviewing the session, also check: did Claude present multi-faceted responses (proposals, options, findings) in a format that let the user reference specific items? If not, note it — the standard is numbered top-level items with lettered sub-points so the user can reply "1b" or "approve 2 and 3".

---

### 3. Cross-check proposals against memory

Before presenting any proposal, read the relevant memory and feedback files and verify the proposal doesn't contradict an established preference. If it does, either drop it or explicitly flag the tension and explain why overriding the preference is warranted. A proposal that conflicts with existing memory without acknowledging it will require a correction round — which is exactly the friction this skill exists to eliminate.

---

### 4. Wait for approval

Present all proposals together. Wait for the user to approve or reject each one before making any changes.

---

### 5. Apply approved changes

- **Creating** new skills: use the `/new-skill <name>` skill (handles write, commit, and symlink)
- **Updating** existing skill files: edit directly with the Edit tool — do NOT use `/new-skill` (it creates an unnecessary symlink). Then commit all changes together below.
- Apply permission additions to `~/.claude/settings.json`
- Commit and push: `cd /Users/davy/dev/claude-skills && git add -A && git commit -m "Session improvements: <summary>" && git push`
- Update memory files and `MEMORY.md` index
