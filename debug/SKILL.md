---
name: debug
description: Use when encountering any bug, test failure, or unexpected behavior — spawns multiple parallel investigators with competing hypotheses to find the root cause faster
---

> Adapted from: https://github.com/anutron/ai/blob/main/skills/debug/SKILL.md
> Upstream SHA: ffe7ccc39068c8ce75e0118d2a6304a2de82d85a

# Competing Hypotheses Debugging

Spawn multiple investigators to debug a problem in parallel. Each pursues a different theory and they argue with each other to converge on the root cause faster than a single-agent linear investigation.

## Prerequisites

Agent teams must be enabled in Claude Code settings:

```
CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
```

If agent teams are not enabled, report: "Agent teams required. Add `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` to your Claude Code env settings." and stop.

## Arguments

- `$ARGUMENTS` - Required: description of the bug, failing test, error message, or unexpected behavior

If no arguments are provided, ask the user what they're debugging.

## Context

- Current branch: !`git branch --show-current`
- Git status: !`git status --short`
- Project root: !`pwd`
- Recent commits: !`git log --oneline -10`
- Test files: !`find . -maxdepth 5 \( -name "*.test.ts" -o -name "*.test.tsx" -o -name "*.spec.ts" \) 2>/dev/null | head -10`

## Overview

You are the **lead investigator** coordinating a debugging team. Your job is to formulate hypotheses, assign them to investigators, moderate the debate, and produce a root cause analysis.

**Problem to debug:** $ARGUMENTS

You do NOT investigate yourself. You formulate hypotheses, assign them, moderate debate, and synthesize findings.

**Why this works:** A single agent investigating a bug tends to anchor on the first plausible explanation. Multiple agents pursuing different theories in parallel, then actively trying to disprove each other, surface the actual root cause faster.

---

## Phase 0: Setup

1. **Analyze the problem** from `$ARGUMENTS` and context above.

2. **Formulate 3 hypotheses** — each should be:
   - Plausible given the symptoms
   - Distinct from the others (different root causes)
   - Testable (there's a way to prove or disprove it)

   Present the 3 hypotheses before spawning investigators.

3. **Create the team:**
   ```
   TeamDelete() — ignore if no existing team
   TeamCreate(team_name: "debug-session", description: "Debug: {brief summary}")
   ```

4. **Create tasks** with TaskCreate:
   - "Investigate hypothesis 1: {brief}" — for investigator-1
   - "Investigate hypothesis 2: {brief}" — for investigator-2
   - "Investigate hypothesis 3: {brief}" — for investigator-3

5. **Spawn 3 investigators** in a single message. Use `model: "sonnet"` for all.

---

## Phase 1: Investigation

Send each investigator their assignment via SendMessage:

```
PROBLEM: {full problem description}

YOUR HYPOTHESIS: {the specific hypothesis assigned}

OTHER HYPOTHESES BEING INVESTIGATED:
- investigator-1: {hypothesis 1}
- investigator-2: {hypothesis 2}
- investigator-3: {hypothesis 3}

INSTRUCTIONS:
1. Explore the codebase for evidence FOR your hypothesis
2. Also look for evidence AGAINST it — be honest
3. Try to reproduce the bug if possible
4. Check git history for recent changes related to your hypothesis
5. Share findings with ALL other investigators:
   - Message each one with your evidence
   - Explain what you found and how it supports or undermines your theory
6. Read and respond to other investigators' findings:
   - If their evidence contradicts your hypothesis, acknowledge it
   - If you can poke holes in their theory, do so with evidence
   - If you become convinced another hypothesis is correct, say so

After investigation and debate, message me (lead) with:
- Verdict: CONFIRMED, DISPROVED, or INCONCLUSIVE
- Key evidence (file paths, line numbers, reproduction steps)
- Whether you now support a different hypothesis

Mark your task completed.
```

Wait for all investigators to report. Allow time for peer debate.

---

## Phase 2: Convergence

```
IF investigators converged on a single root cause:
  → Proceed to Phase 3

IF two or more theories remain plausible:
  → Send tiebreaker prompt (see below)

IF all hypotheses disproved:
  → Formulate new hypotheses from evidence gathered, return to Phase 1
     (max 2 total rounds)
```

### Tiebreaker prompt (send to all remaining investigators):

```
REMAINING HYPOTHESES:
{list the hypotheses still standing with evidence for each}

INSTRUCTIONS:
1. Design a SPECIFIC test that would distinguish between these theories
   (If A is correct, X should happen. If B is correct, Z should happen instead.)
2. Run that test or trace through the code to determine the outcome
3. Share results with all other investigators
4. Message me with your final verdict
```

---

## Phase 3: Fix

Once root cause is identified:

1. **Assign fix tasks** via TaskCreate (mark investigator-2 and investigator-3 tasks blocked by fix):
   - "Implement fix for: {root cause}" — assign to the investigator who identified it
   - "Verify fix" — second investigator, blocked by fix task
   - "Check for regressions" — third investigator, blocked by fix task

2. **Send fix request** to the assigned investigator:

```
ROOT CAUSE CONFIRMED: {description}

Implement a fix. Keep it minimal — fix the bug, nothing more.
Run targeted tests for the packages you changed (cd backend && npm test -- --run, or cd frontend && npm test -- --run).
Commit with message: "Fix: {short description}"
When done, message me with your changes.
```

3. **Send verification request** to second investigator after fix lands.

4. **Send regression check** to third investigator: run full test suite in both packages.

Wait for all three to complete.

---

## Phase 4: Shutdown and Summary

1. Shut down all investigators via SendMessage with "Investigation complete."
2. Clean up team with TeamDelete.
3. Produce investigation report:

```markdown
## Debug Report

### Problem
{original description}

### Root Cause
{1-2 sentences}

### Investigation

| Hypothesis | Verdict | Key Evidence |
|------------|---------|--------------|
| {hypothesis 1} | CONFIRMED/DISPROVED | {1-line} |
| {hypothesis 2} | CONFIRMED/DISPROVED | {1-line} |
| {hypothesis 3} | CONFIRMED/DISPROVED | {1-line} |

### Fix Applied
| File | Change |
|------|--------|
| path/to/file | brief description |

### Verification
- Tests pass: YES/NO
- Regression risk: low/medium/high
```

---

## Agent Briefing (use this prompt for all 3 investigators)

```
You are an INVESTIGATOR on a debugging team. Your job is to find root causes.

YOUR APPROACH:
1. Gather evidence FOR and AGAINST your hypothesis
2. Share findings with other investigators via direct messages
3. Challenge their theories with evidence; accept challenges to yours
4. If your theory is disproved, pivot — help validate or disprove others
5. Be specific: cite file paths, line numbers, reproduction steps
6. After investigation, you may be asked to implement a fix, verify it, or check regressions

The goal is TRUTH, not winning. Abandon your hypothesis the moment evidence disproves it.

Always use TaskUpdate to mark tasks completed.
```

---

## Failure Handling

| Failure | Action |
|---------|--------|
| Agent fails to spawn | Retry once; proceed with 2 investigators if needed |
| All hypotheses disproved (round 1) | Formulate new hypotheses from gathered evidence |
| All hypotheses disproved (round 2) | Report evidence gathered; suggest manual investigation |
| Investigators can't converge | Lead makes judgment call based on evidence weight; note uncertainty |
| Team creation fails | Report the prerequisite error and stop |
