---
name: skill-tdd
description: Test-driven development for Claude skills. Generates scenario files, runs behavioral tests against isolated agent sessions, and integrates with /improve for regression detection.
---

# Skill TDD

Behavioral test framework for Claude Code skills. Each skill gets a `tests/scenarios.md` describing concrete input/expected-behavior pairs derived from past friction. A test runner invokes the skill in an isolated agent session per scenario and evaluates the output against assertions.

## Usage

- `/skill-tdd generate <skill-name>` — generate or refresh `tests/scenarios.md` for a skill from its SKILL.md + session memory
- `/skill-tdd run <skill-name>` — run all scenarios for a skill, report pass/fail
- `/skill-tdd run --all` — run tests for every skill that has a `tests/scenarios.md`
- `/skill-tdd retrofit` — generate scenarios for all skills that don't have one yet

---

## Run mode

### 1. Find the skill

Locate the skill directory:
1. Check `/Users/davy/dev/claude-skills/<name>/` (public)
2. Check `/Users/davy/dev/claude-skills-private/<name>/` (private)

Read `SKILL.md` and `tests/scenarios.md`. If `tests/scenarios.md` is missing, say so and offer to run `generate` mode first.

### 2. Run each scenario

For each scenario in `tests/scenarios.md`, invoke the `Agent` tool with a subagent_type of `general-purpose`. The agent prompt must:

1. Paste the full SKILL.md content as context
2. Include the scenario's **Input** and **Context** sections verbatim
3. Instruct the agent: *"You are executing the `<skill-name>` Claude Code skill. Follow the SKILL.md instructions exactly as if this were a live user session. Use real tools where the scenario requires it. When done, output your work."*

Run all scenarios in parallel when they are independent (no shared file state). Run sequentially when scenarios share fixture files.

### 3. Evaluate each result

For each scenario, read the agent's output and check every assertion in the **MUST** and **MUST NOT** lists. Evaluate semantically — assertions are descriptions of behavior, not string matches. Mark each assertion:

- `✓ PASS` — satisfied
- `✗ FAIL` — violated (quote the specific output that caused the failure)
- `? SKIP` — could not evaluate (e.g., scenario required a live external service)

A scenario PASSES only if all MUST assertions pass and all MUST NOT assertions pass.

### 4. Report

Output a table:

```
Skill: <name>          Scenarios: N    Passed: N    Failed: N
────────────────────────────────────────────────────────────
✓  scenario-name        all assertions satisfied
✗  scenario-name        FAIL: "..." violated assertion: "MUST NOT include graduation years"
────────────────────────────────────────────────────────────
```

For each failure, include:
- The failing assertion
- The quoted output excerpt that violated it
- A one-line diagnosis of what rule the skill should enforce

### 5. On failure: propose fixes

For each failing scenario, identify which instruction in SKILL.md is missing, ambiguous, or contradicted by the output. Draft a specific edit to SKILL.md that would prevent the failure. Group all proposed edits and present them together.

If invoked from `/improve`, apply fixes automatically and re-run failed scenarios. Escalate if still failing after one fix cycle.

---

## Generate mode

### 1. Read sources

Read in parallel:
- The skill's `SKILL.md`
- All memory files in `~/.claude/projects/*/memory/` that reference this skill or its domain
- `~/.claude/memory/MEMORY.md` (global)
- The current session transcript if available

### 2. Identify friction patterns

Look for:
- Corrections the user made to the skill's output (explicit or implicit)
- Cases where the skill routed something wrongly (silently disqualified, skipped, or misclassified)
- Edge cases mentioned in SKILL.md that aren't tested
- Rules in memory feedback files that correspond to this skill

### 3. Write scenarios.md

Create `tests/scenarios.md` in the skill directory. For each friction pattern found, write one scenario using this format:

```markdown
## Scenario: <kebab-case-name>
**Rationale:** <one sentence — what past friction or rule this scenario guards against>

### Input
<what the user would provide — URL, pasted text, command arguments, or description>

### Context
<any background files, state, or facts the agent needs; embed minimal fixture data inline rather than referencing external files>

### MUST
- The output [specific observable behavior]
- The output [another assertion]

### MUST NOT
- The output [behavior that indicates failure]

---
```

Write at least 4 scenarios per skill. Include at least one happy-path scenario and at least one edge-case that caused real past friction.

### 4. Save

Write the file. Report how many scenarios were generated and which friction patterns they cover.

---

## Retrofit mode

Read all skill directories in both repos. For each skill without a `tests/scenarios.md`, run generate mode for it. Process skills in parallel.

---

## Scenarios.md format reference

```markdown
# Scenarios: <skill-name>

Each scenario is run in an isolated agent session with the skill's SKILL.md as instructions.

---

## Scenario: <kebab-name>
**Rationale:** <why this matters>

### Input
<what to give the agent as the user request>

### Context
<setup info — fixture data, file state, background facts>

### MUST
- The output contains/does/includes [X]
- Claude stops/proceeds/routes [X] correctly

### MUST NOT
- The output contains [Y]
- Claude [performs forbidden action]

---
```
