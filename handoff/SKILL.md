---
name: handoff
description: Generate a structured handoff prompt capturing current session context and copy to clipboard. Use when starting a /clear or passing context to another agent.
---

> Adapted from: https://github.com/anutron/ai/blob/main/skills/handoff/SKILL.md
> Upstream SHA: 5a88f13108fe3e62444cfc4ac315475362032942

# Context Handoff

Generate a structured prompt capturing the current conversation context so it can be pasted into another agent thread or saved for session continuity.

## Arguments

- `$ARGUMENTS` - Optional: what to emphasize or who the target is

## Context

- Repo: !`git rev-parse --show-toplevel 2>/dev/null | xargs basename 2>/dev/null | head -1`
- Branch: !`git branch --show-current`
- Recent commits: !`git log origin/HEAD..HEAD --oneline 2>/dev/null | head -15`
- Uncommitted changes: !`git status --short 2>/dev/null | head -20`
- Changed files vs base: !`git diff --name-only origin/HEAD..HEAD 2>/dev/null | head -30`

## Instructions

Review the full conversation history and synthesize a handoff prompt.

### Step 1: Generate the Handoff Document

Use this format, omitting empty sections:

```
## Handoff: [brief title]

### Background
[1-3 sentences on what was being worked on and why]

### What Was Done
- [Completed work with specific file paths]

### Current State
[Branch state, what is working, what is not]

### Key Decisions
- [Decision]: [Rationale]

### Remaining Work
- [ ] [Specific actionable items]

### Important Context
- [Gotchas, constraints, or patterns the next agent needs]
- [Specific file paths, function names, code patterns]

### Files to Read First
- [Ordered list of files to get up to speed]
```

Keep it concise but complete enough that the receiving agent can continue without re-discovering context.

### Step 2: Present and Copy to Clipboard

Print the handoff inside a fenced code block so the user can review it. Then copy it to the clipboard:

```bash
cat <<'HANDOFF_EOF' | pbcopy
[the full handoff document content]
HANDOFF_EOF
```

Tell the user: "Copied to clipboard. Run `/clear` then paste to start a fresh session with full context."

### Step 3: Save to Memory

Save a brief summary to the project memory directory. The project slug is the path-encoded form of the working directory (e.g. `~/.claude/projects/-Users-davy-dev-tech-bridge/memory/`). Write `handoff_<YYYY-MM-DD>.md` capturing what was handed off and what's remaining — so the next session can recover context even if the clipboard is cleared.
