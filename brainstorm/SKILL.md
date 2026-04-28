---
name: brainstorm
description: Use before any non-trivial feature or design decision — explores intent, surfaces assumptions, proposes approaches, writes a design doc and implementation plan, then hands off to /implement.
user-invocable: true
---

> Adapted from: https://github.com/anutron/ai/blob/main/skills/brainstorm/SKILL.md
> Upstream SHA: 9278ada3e6092a4a365f16707ccc093143de8895

# Brainstorm: From Idea to Implementation Plan

Turn ideas into fully-formed designs and strategic implementation plans. Two phases — brainstorm (design the solution) and planning (map the execution) — with one review gate before any code is written.

## Arguments

- `$ARGUMENTS` - Optional: description of the feature or idea to develop

## Hard Gate

Do not write any code or take any implementation action until you have presented a design and the user has approved it. This applies regardless of perceived simplicity. The design can be short, but it must be presented and approved.

---

# Phase 1: Brainstorm

## Step 1: Read project context

Before asking questions, read:
1. `CLAUDE.md` — project instructions, conventions, stack
2. Relevant source files as needed to understand current state

## Step 2: Surface assumptions

Present your understanding before asking questions:

> "Based on what I see, here are my assumptions:
> 1. [assumption]
> 2. [assumption]
>
> Correct me now or I'll proceed with these."

This front-loads alignment and eliminates most clarifying questions.

## Step 3: Size check

For genuinely small changes (localized, well-understood, low risk), ask:
- "This looks like a small change. Want the full brainstorm, or should I just confirm the approach and go?"
  - **Full brainstorm** — continue
  - **Quick confirm** — skip to a brief approach confirmation, then Phase 2

Only offer this for small changes. Medium and large always get the full process.

## Step 4: Clarifying questions

Ask one at a time. Prefer multiple choice. Focus on purpose, constraints, and success criteria.

If the request spans multiple independent subsystems, flag this and help decompose into sub-projects first — each gets its own brainstorm.

## Step 5: Norms check

Before proposing approaches, ask: "What does this kind of problem usually look like, and what's the standard professional approach?"

If the user's framing skips a well-established solution — hashing for credentials, parameterized queries for SQL, env vars for secrets, tests for behavioral code — surface it as an option. State the norm and why it exists, then let the user choose knowingly.

Skip when the problem space has no relevant industry norm.

## Step 6: Silent pre-mortem

Silently assess: what could go wrong? How big is the blast radius?

- **Small** (localized, easy to revert, no data risk) — proceed without comment
- **Large** (data loss risk, breaking change, hard to revert) — surface the risk and ask how to proceed

## Step 7: Propose approaches

Present 2-3 approaches with tradeoffs. Lead with your recommended option and defend it. Be opinionated. YAGNI ruthlessly.

## Step 8: Present design in sections

Present the design and validate each section before moving on. Scale to complexity — a few sentences if simple, more if nuanced. Cover as relevant: architecture, data flow, components, error handling, testing strategy.

Design for isolation: each unit should have one clear purpose and be understandable independently.

**Working in existing codebases:**
- Follow existing patterns
- Apply Chesterton's Fence: understand why existing code exists before changing it
- Do not propose unrelated refactoring

## Step 9: Write brainstorm doc

Save to `plans/<YYYY-MM-DD>-<topic>/brainstorm.md`. Commit immediately.

Self-review before committing:
1. Placeholder scan: any TBD, TODO, incomplete sections? Fix them.
2. Internal consistency: do sections contradict each other?
3. Scope check: is this focused enough for one implementation plan?
4. Ambiguity check: could any requirement be interpreted two ways? Pick one.

Offer to open in plannotator for review:
- "Brainstorm doc written. Want to review it in plannotator before I move to planning?"
- If yes: invoke `plannotator:plannotator-annotate` on the doc. Address annotations, then proceed.
- If no: proceed to Phase 2.

---

# Phase 2: Planning

## Step 1: Read the brainstorm doc

Read the written brainstorm doc as input — not conversation history.

## Step 2: Write the implementation plan

Save to `plans/<YYYY-MM-DD>-<topic>/plan.md` (same directory). English only, no code. Tells the implementing agent what to build and in what order.

Structure:

```markdown
# <Feature Name> — Implementation Plan

**Goal:** <Restated from the brainstorm>
**Design doc:** `<path to brainstorm.md>` — Read this first for architecture and design decisions.

**Assumptions and boundaries:**
- What's in scope
- What's not in scope
- What we're relying on

## Stages

### Stage 1: <First vertical slice>
<What this stage delivers. What files/areas it touches. Done criteria.>

### Stage 2: <Next vertical slice>
**Depends on:** Stage 1
<Same structure. Each stage is a vertical slice delivering one complete path.>
```

Key principles:
- **Vertical slices** — each stage delivers one complete end-to-end path, not a horizontal layer
- **Dependencies** — note which stages depend on which, so parallel stages are clear
- **Chesterton's Fence** — understand existing code before changing it

Commit the plan immediately.

## Step 3: Present plan for review

Call `EnterPlanMode` and present the plan body. This is the one real review gate — no code is written until the user exits plan mode.

If plan review produces feedback that changes the design, exit plan mode, update the brainstorm doc, amend the plan, then re-enter plan mode.

## Step 4: Execution handoff

After plan approval (`ExitPlanMode`):

- Create a GitHub issue for the feature using `/issue` — include the plan as the issue body so `/implement` can find it
- Tell the user: "Plan approved. Run `/implement <issue-number>` to execute."
