# Scenarios: adapt-skills

Each scenario is run in an isolated agent session with the skill's SKILL.md as instructions.

---

## Scenario: no-pre-screening-by-name
**Rationale:** Skill names are often misleading. "magic", "deploy", "utils" could contain valuable workflows. Pre-screening by name causes useful skills to be skipped without evaluation. The only way to evaluate fit is to read the content.

### Input
/adapt-skills https://github.com/example/ai-toolbox

### Context
The repo contains these skill files in .claude/commands/:
- `magic/SKILL.md` — content: a detailed deploy-and-monitor workflow for production releases
- `cleanup/SKILL.md` — content: removes stale branches and worktrees after merges
- `snapshot/SKILL.md` — content: creates a timestamped project state snapshot for rollback

All three have non-obvious names relative to their content.

### MUST
- Fetch and read the content of all three skill files (magic, cleanup, snapshot)
- Evaluate each based on its content, not its name
- Include all three in the analysis/recommendations output

### MUST NOT
- Skip any skill based on its name alone
- Assume a skill's purpose without reading it

---

## Scenario: batch-adaptation-single-plannotator-session
**Rationale:** Calling /new-skill N times opens N plannotator sessions — one per skill — which is disruptive and inconsistent. Batch adaptation must write all files then invoke plannotator once.

### Input
/adapt-skills https://github.com/example/skills-repo

User selects 3 skills to adapt: "deploy", "audit", "standup"

### Context
The user has confirmed they want to bring in all three skills.

### MUST
- Write all three SKILL.md files with the Write tool before invoking any review
- Invoke plannotator-review exactly once (after all files are written)
- Commit all three skills in a single git commit

### MUST NOT
- Call /new-skill for any of the adapted skills
- Invoke plannotator-review once per skill (N separate sessions)
- Commit each skill separately

---

## Scenario: credit-block-in-adapted-skill
**Rationale:** Adapted skills need to track their upstream source so /adapt-skills update can detect drift. Without the credit block (URL + SHA), update mode can't work.

### Input
/adapt-skills https://github.com/anutron/ai

User selects the "pr-respond" skill to adapt.

### Context
mcp__github__get_file_contents returns:
- content: [the skill file content]
- html_url: "https://github.com/anutron/ai/blob/main/skills/pr-respond/SKILL.md"
- sha: "abc123def456"

### MUST
- The written SKILL.md includes a line: `> Adapted from: https://github.com/anutron/ai/blob/main/skills/pr-respond/SKILL.md`
- The written SKILL.md includes a line: `> Upstream SHA: abc123def456`
- Both lines appear near the top of the file (after any frontmatter)

### MUST NOT
- Omit either the URL or the SHA from the credit block
- Use the API URL instead of the html_url

---

## Scenario: update-mode-shows-diff-before-adopting
**Rationale:** Update mode must show the user what changed upstream before applying anything. Silent auto-updates to skills could introduce unwanted behavior changes.

### Input
/adapt-skills update

### Context
The local pr-respond/SKILL.md contains:
`> Upstream SHA: abc123`

mcp__github__get_file_contents returns the current file with SHA: `xyz789` (different from stored `abc123`).

The upstream content has one changed section: added a new step about handling "Request changes" review state.

### MUST
- Detect the SHA mismatch and report pr-respond as "Changed" in the status table
- Fetch the upstream content
- Present a clear diff showing what changed (the new "Request changes" step)
- Ask the user whether to adopt the change before applying it

### MUST NOT
- Auto-apply upstream changes without showing the diff
- Update the local SHA without user confirmation
- Skip skills with mismatched SHAs

---

## Scenario: update-mode-reports-up-to-date-skills
**Rationale:** The status table should confirm which skills are current, not just list changed ones. This gives the user confidence their adapted skills haven't drifted.

### Input
/adapt-skills update

### Context
Two adapted skills:
1. pr-respond — local SHA: `abc123`, upstream SHA: `abc123` (same)
2. devils-advocate — local SHA: `def456`, upstream SHA: `def456` (same)

### MUST
- Display both skills in the status table with "Up to date" status
- No diff or adopt prompt for either

### MUST NOT
- Report any skill as changed when SHAs match
- Skip displaying skills that are current
