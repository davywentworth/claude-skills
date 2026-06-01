---
name: improve
description: This skill should be used at the end of a session to audit friction points, propose skill and permission improvements, and apply changes to the ecosystem. Examines how the user interacted with Claude and proposes concrete improvements across skills, permissions, memory, and workflow.
user-invocable: true
effort: high
---

End-of-session review to make the user more efficient in future sessions. Examines how the user interacted with Claude and proposes concrete improvements across skills, permissions, memory, and workflow.

## Goal

Make interactions simpler without sacrificing power. The user should be able to say less and get more — shorter inputs, fewer approvals, less back-and-forth — while Claude handles the same quality and depth of work. Look for anything that slowed the user down, required extra clarification, needed manual intervention, or could have been inferred. Every approval prompt, correction, repeated instruction, or workaround is a candidate for elimination.

---

## Steps

### 0. Read the ecosystem inventory

Before analyzing the session, read all four ecosystem artifacts so you have a current picture of what exists:

1. `~/.claude/CLAUDE.md` — global preferences
2. `~/.claude/settings.json` — permissions
3. `SKILL.md` files — see scope rules below
4. `~/.claude/projects/<current-project>/memory/MEMORY.md` and all memory files it references

**Reading mechanism:** Always use the `Read` tool for every file. Never use `cat`, `head`, `tail`, or bash glob loops — those start with shell keywords that match no allowed Bash prefix and will prompt for permission. Multiple `Read` calls in a single parallel message are the correct pattern.

**SKILL.md read scope — two passes:**

*Pass 1 (always):* Read the SKILL.md for every skill explicitly invoked or modified this session (check the transcript). These are the only skills guaranteed to need review.

*Pass 2 (targeted sample):* Read 3 additional skills chosen as: the 3 skills whose names come first alphabetically among those NOT already read in Pass 1. This provides a lightweight drift check without reading all 26 files every session.

Skip reading skills you already have in context from this session — do not re-read unchanged files.

Any proposal that touches one artifact must be checked against all others for consistency.

---

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
For each skill with actionable feedback: read the current file fresh by following the symlink at `~/.claude/commands/<name>` to get the real path on disk (do not rely on context — it may be stale). Draft the proposed change and explain what friction it removes. Also check all skills for:
- **Hardcoded project-specific values**: absolute paths, repo names, usernames, port numbers, or org names — flag any that should be derived dynamically (e.g. `git rev-parse`, `gh repo view`, `pwd`) or parameterised.
- **Missing skill delegation**: any skill that makes code changes and commits without first invoking `/review` is missing a quality gate. Flag it and propose adding the invocation before the commit step.
- **Cross-skill pattern consistency**: if any skill was updated this session to change a tool or pattern (e.g. `gh` → MCP, one library → another), grep all other skills for the old pattern and flag any that weren't updated in the same pass.
- **Token efficiency audit** (global directive: favor cost reduction): flag any skill that exhibits these patterns:
  - **Repeated context injection** — a large file or block pasted verbatim into every parallel agent's prompt (e.g. rules.md injected into 11 agents). Fix: extract the relevant subset once in the orchestrator, inject the compressed form.
  - **1:1 agent-per-item spawning** — one verification/research/fetch agent spawned per candidate or item. Fix: batch 3–5 items per agent.
  - **No-checkpoint long-running skills** — skills that spawn many agents and write nothing to disk until the very end. If the session ends mid-run, all work is lost. Fix: write a staging file after each major phase so `/skill-name continue` can resume.
  - **Full file reads in every agent** — agents re-reading the same large file (README, KB, rules) independently. Fix: read once in the orchestrator, summarize or extract the relevant slice, pass that to agents.

#### 2b. New skills
For patterns that came up ad-hoc and would benefit from a reusable skill: propose a name, one-line description, and why it reduces friction based on this session.

#### 2c. Permission additions

**Step 1 — Settings gap audit (do this first, before looking at the log):**

Read both settings files:
- `~/.claude/settings.json` — global permissions
- `<project-root>/.claude/settings.local.json` — project-level permissions (check if this file exists)

In a project context, `settings.local.json` is what's enforced — rules in `settings.json` do NOT automatically apply. For every rule in `settings.json`, check whether an equivalent rule exists in `settings.local.json`. Any rule that's in the global file but missing from the project file is a gap that will cause permission prompts. List every such gap explicitly.

**Step 2 — Permission log audit:**

Use the `Read` tool on `~/.claude/permission-log.jsonl` — do NOT use `python3 -c`, `cat`, or any bash command to read or parse it. `python3 -c` executes arbitrary inline code and will always prompt regardless of allowlist rules; `cat` requires `Bash(cat:*)`. The Read tool is the only safe mechanism here.

After reading, scan the entries manually. The log uses pretty-printed multi-line JSON objects. Look for entries where `permission_suggestions` is non-empty — those are real prompts. Critical: `"destination": "session"` in a suggestion means the user was prompted and approved a one-time session grant (i.e., they saw a dialog). Only `"destination": "settings"` indicates a permanent rule was already in place. Treat any `"destination": "session"` entry as user friction.

Cross-reference each log entry with the settings gap list from Step 1. Entries that match a missing rule confirm the gap caused a real prompt.

**Step 2b — Transcript frequency audit:**

Grep the current session transcript for permission prompt patterns (look for phrases like "needs your permission", "would like to", "Allow", tool-call denial messages, and repeated identical Bash/Read/MCP calls that appear more than once). Group by resource (file path, command prefix, or MCP tool name) and count occurrences. Any resource that was prompted **more than once** in the same session is a settings gap — the user had to approve the same thing multiple times, which is pure friction. Add these to the gap list alongside any found in Step 2.

**Step 3 — Propose fixes:**

For each confirmed gap or prompted call, assess whether it's safe to auto-approve:
- Safe globally: read-only operations (`ls`, `grep`, `cat`, `tail`), MCP read calls, posting comments
- Case-by-case: destructive operations, force pushes, branch deletion

Propose additions to `permissions.allow` in `settings.local.json`. Syntax:
- **`Bash`**: prefix syntax — `Bash(ls:*)` — matches commands starting with that string
- **`Read`**: glob syntax — `Read(/some/path/**)` — matches file paths

Prefer directory-scoped rules over broad ones when the operation is only safe in a specific location.

#### 2d. Memory updates
For any new patterns, preferences, or corrections that should persist: determine whether they belong in `~/.claude/CLAUDE.md` (cross-project, applies everywhere) or the project memory files (specific to this repo). Propose additions or updates accordingly. If something is now covered by a skill or already in the global CLAUDE.md, trim the redundant entry.

When reviewing the session, also check: did Claude present multi-faceted responses (proposals, options, findings) in a format that let the user reference specific items? If not, note it — the standard is numbered top-level items with lettered sub-points so the user can reply "1b" or "approve 2 and 3".

---

### 3. Cross-check proposals against memory

Before applying any change, read the relevant memory and feedback files and verify the proposal doesn't contradict an established preference. If it does, either drop it or explicitly flag the tension and explain why overriding the preference is warranted. A proposal that conflicts with existing memory without acknowledging it will require a correction round — which is exactly the friction this skill exists to eliminate.

---

### 4. Run skill tests

Before applying any skill changes, run behavioral tests for every skill that either (a) has a proposal in this session or (b) has a `tests/scenarios.md` file.

For each skill with a `tests/scenarios.md`:
1. Invoke `/skill-tdd run <skill-name>` — spawns isolated agent sessions per scenario and evaluates assertions
2. Record pass/fail results

**On failures:**
- If the failing scenario corresponds to a friction pattern already in the proposal: apply the proposed fix, then re-run the failing scenario
- If the failing scenario is a regression (not addressed by any proposal): add a fix to the proposal before applying
- Re-run failing scenarios after each fix; escalate if still failing after one fix cycle (report: scenario name, assertion violated, what was tried)

**On all-pass:** proceed to apply changes as normal. Note in the commit message which tests ran and passed.

Skip this step if no skills have a `tests/scenarios.md` yet — but note in the output that test coverage is missing.

---

### 5. Apply changes

Present findings, then immediately apply all changes without waiting for approval — skill files are committed to git and trivially reversible. The user can push back after the fact if anything is wrong.

- **Creating** new skills: use the `/new-skill <name>` skill (handles write, commit, and symlink)
- **Updating** existing skill files: edit directly with the Edit tool — do NOT use `/new-skill` (it creates an unnecessary symlink). Then commit all changes together below.
- Apply permission additions to the project's `.claude/settings.local.json` (required for the permission to take effect in project context). If the permission is useful across all projects, also add it to `~/.claude/settings.json`.
- **Run `/fewer-permission-prompts`** after applying manual permission additions — it scans transcripts for read-only Bash and MCP tool calls missed by the manual audit and adds them to the allowlist. This is especially important if any scheduled or unattended agents ran this session, since they can be interrupted by prompts the user never saw.
- Update `~/.claude/CLAUDE.md` for cross-project preferences
- Commit and push: for each skill repo with changes, use `git -C <repo-path>` for every git operation — do NOT use `cd <repo-path> && git` since Claude Code splits `&&` chains before permission matching and the cd-prefixed form never matches allowlist rules. Run: `git -C <repo-path> add -A && git -C <repo-path> commit -m "Session improvements: <summary>" && git -C <repo-path> push` — keep the message a single line; heredocs inside `&&` chains cause EOF parse errors
- Update project memory files and `MEMORY.md` index
