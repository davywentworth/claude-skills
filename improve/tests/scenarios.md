# Scenarios: improve

Each scenario is run in an isolated agent session with the skill's SKILL.md as instructions.

---

## Scenario: settings-gap-between-global-and-local
**Rationale:** In a project context, settings.local.json is what's enforced — global settings.json rules do NOT automatically apply. A rule in global but missing from local causes permission prompts every session. Improve must audit this gap.

### Input
/improve

### Context
Session just ended. ~/.claude/settings.json contains:
```json
{
  "permissions": {
    "allow": ["Bash(ls:*)", "Read(/Users/davy/research/**)"]
  }
}
```

Project .claude/settings.local.json contains:
```json
{
  "permissions": {
    "allow": ["Bash(git status:*)"]
  }
}
```

No permission prompts appeared in this session.

### MUST
- Flag that Bash(ls:*) and Read(/Users/davy/research/**) are in global settings but missing from settings.local.json
- Propose adding both to settings.local.json
- Identify this as a potential source of permission prompts in project context

### MUST NOT
- Report "no settings gaps" when global rules aren't mirrored in the project file
- Skip the settings gap audit step

---

## Scenario: repeated-permission-prompt-becomes-permanent-rule
**Rationale:** A tool call the user approved once should not prompt again in the same session or future sessions. If a tool was prompted more than once, that's a clear signal it should be permanently allowed.

### Input
/improve

### Context
The session permission log shows:
```
{"behavior": "allow", "destination": "session", "tool": "Read", "path": "/Users/davy/research/job-search/README.md"}
{"behavior": "allow", "destination": "session", "tool": "Read", "path": "/Users/davy/research/job-search/README.md"}
{"behavior": "allow", "destination": "session", "tool": "Read", "path": "/Users/davy/research/job-search/rules.md"}
```

(Two approvals for the same Read path — user was prompted twice.)

### MUST
- Identify that Read(/Users/davy/research/job-search/README.md) was prompted more than once
- Propose adding Read(/Users/davy/research/**) to settings.local.json as a permanent rule
- Explain that destination: "session" means the user was prompted (not auto-approved)

### MUST NOT
- Interpret "destination: session" as auto-approved
- Propose nothing because "the user already allowed it"

---

## Scenario: cross-skill-pattern-consistency-check
**Rationale:** When a pattern is updated in one skill (e.g., gh CLI → mcp__github__ tools), all other skills using the old pattern must be updated in the same pass. The user should not discover inconsistencies session-by-session.

### Input
/improve

### Context
This session updated the `pr` skill to replace `gh pr create` with `mcp__github__create_pull_request`.

Other skills in /Users/davy/dev/claude-skills/:
- implement/SKILL.md: contains `gh pr create --title "..." --body "..."`
- pr-respond/SKILL.md: contains `gh pr list --json number`
- review/SKILL.md: no gh commands

### MUST
- Detect that `implement` and `pr-respond` still contain the old `gh pr` pattern
- Flag both as needing updates for cross-skill pattern consistency
- Propose specific edits to each skill

### MUST NOT
- Report consistency as clean when other skills still use the old pattern
- Only check the skill that was directly updated this session

---

## Scenario: skill-update-uses-edit-not-new-skill
**Rationale:** /new-skill creates a new symlink and directory structure. Calling it on an existing skill creates an unnecessary symlink. Existing skills must be updated with the Edit tool directly.

### Input
/improve

### Context
The session revealed that the `debug` skill is missing a step: it doesn't invoke /review before committing. The fix is to add one line to debug/SKILL.md.

### MUST
- Edit /Users/davy/dev/claude-skills/debug/SKILL.md directly with the Edit tool
- Add the missing /review invocation before the commit step

### MUST NOT
- Call /new-skill debug (would create a duplicate symlink)
- Create a new SKILL.md file instead of editing the existing one

---

## Scenario: proposal-uses-numbered-list-format
**Rationale:** Multi-faceted proposals must use numbered top-level items with lettered sub-points. This lets the user reply "1b" or "approve 2 and 3" without ambiguity.

### Input
/improve

### Context
The session produced findings across three categories:
1. One skill update (add step to debug)
2. One new skill proposal (skill-health-check)
3. Two permission additions

### MUST
- The proposal uses numbered top-level items (1, 2, 3...)
- Sub-points within an item use lettered lists (a, b, c)
- The user can reference specific items with "1b" style replies

### MUST NOT
- Present findings as a flat bullet list
- Use only bullets with no numbered items
- Mix numbering conventions within the proposal

---

## Scenario: transport-migration-consumer-audit
**Rationale:** After a transport/abstraction migration, consumers that rely on the old implementation's behavior (error semantics, reconnect events, message ordering) must all be audited — not just the module that changed. Stopping at the interface boundary misses behavioral dependencies.

### Input
/improve

### Context
This session replaced `useServer.js` (WebSocket transport) with a new SSE-based implementation. The hook interface `{ state, message, send }` was preserved unchanged.

Other files that import from useServer.js:
- App.jsx — uses `state === 'error'` to show a reconnect banner
- WireframeView.jsx — uses `message` to update annotations
- StatusBar.jsx — uses `state` for connection indicator color

### MUST
- Identify all consumers of useServer.js (App.jsx, WireframeView.jsx, StatusBar.jsx)
- Flag that each consumer's behavioral assumptions about the old transport must be audited
- For App.jsx specifically: flag that `state === 'error'` semantics may differ between WebSocket and SSE

### MUST NOT
- Stop auditing at the useServer.js module boundary
- Assume consumers are fine because the hook interface was preserved
- Only audit the file that directly changed
