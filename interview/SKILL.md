---
name: interview
description: Structured interview-style review of any system, feature, or codebase. Builds an inventory, walks through items one-by-one in small chunks, tracks progress, captures decisions as artifacts. Use when you want to systematically review, audit, or evaluate something collaboratively.
---

> Adapted from: https://github.com/anutron/ai/blob/main/skills/interview/SKILL.md
> Upstream SHA: cfaaca03833a6380d030b0fa741e99a881530190

# Structured Interview Review

Run a collaborative, structured review of any system, feature, or domain. You are the interviewer — present material in small chunks, the user asks questions and makes decisions, and you capture everything into agreed-upon artifacts.

## Arguments

- `$ARGUMENTS` - What to review (e.g. "the permissions system", "our API routes", "the onboarding flow")

**Before asking anything**, check for an existing `*_review/` directory:
```bash
find . -maxdepth 1 -type d -name "*_review"
```
If one exists with `inventory.md` and `discussion.log`, read both, present the inventory with current status, and resume from the first pending item. Do not ask what to review.

If no review is in progress and no arguments provided, ask: "What would you like to review?"

## Context

- Working directory: !`pwd`
- Git branch: !`git branch --show-current 2>/dev/null | head -1`
- Date: !`date +%Y-%m-%d`

---

## Phase 0: Goal and Process Negotiation

Go through these questions **one at a time**, waiting for a response before moving on.

### 0a. Confirm the goal

Restate the review goal from `$ARGUMENTS` in your own words. Ask: "Is that right, or should I adjust the scope?"

### 0b. Explore and propose an inventory

Explore the codebase or domain to build a proposed inventory of items to review. Present it as a structured list grouped into sections. Ask: "Does this cover everything? Anything to add, remove, or regroup?"

### 0c. Propose the review axes

Based on what you found, propose how each item should be evaluated. Examples:
- Edit / View / None (permission levels)
- Keep / Change / Remove (audit)
- Meets spec / Needs work / Missing (compliance)
- Custom axes the domain suggests

Ask: "How should we evaluate each item? Here is what I would suggest: [proposal]. Or tell me your framework."

### 0d. Negotiate the output format

Propose what the review should produce. Be specific about artifact types. Examples:
- GitHub issues (one per change, with full context — use `/issue` to file them)
- A summary report or audit document saved to `<review-name>_review/report.md`
- A QA checklist per section
- SPEC documents (one per change, detailed enough for implementation without user input)

Present 2-3 options that make sense for the goal, recommend one, and ask: "What should the output be? I'd recommend [X] because [reason]."

The user may choose one, combine several, or specify something else. Whatever they choose becomes the artifact format for the rest of the review.

### 0e. Set up the working directory

Once goal, inventory, axes, and output are agreed:

1. Create a working directory: `<review-name>_review/` in the project root
2. Create `inventory.md` with the agreed inventory, all items marked pending
3. Create `log.sh` — a shell script for appending to a discussion log:

```bash
#!/bin/bash
# Usage: ./log.sh <speaker> "<message>"
LOGFILE="$(dirname "$0")/discussion.log"
echo "" >> "$LOGFILE"
echo "## $1 — $(date '+%Y-%m-%d %H:%M')" >> "$LOGFILE"
echo "" >> "$LOGFILE"
echo "$2" >> "$LOGFILE"
```

4. Make it executable: `chmod +x log.sh`
5. Initialize `discussion.log` with the agreed goal, axes, and output format
6. Create any artifact templates needed

Confirm setup is complete and present the first section.

---

## Phase 1: The Interview

### Section Loop

For each section in the inventory:

1. Present the section header with a progress table showing all items and their status (pending / reviewing / complete)
2. Move to the first pending item

### Item Loop

For each item in the current section:

1. **Present one chunk** of the item. Never present the entire item at once. If there is a natural decomposition, use it. If not, break into 3-5 manageable chunks.

2. **Wait for the user's response.** They may:
   - Ask clarifying questions — answer them
   - Give an instruction ("add X", "change Y", "remove Z") — acknowledge it and create the agreed artifact immediately
   - Say it looks fine — note it and move on
   - Want to dive deeper — present more detail on the current chunk

3. **After the chunk is resolved**, present the next chunk of the same item.

4. **After all chunks for an item are resolved:**
   - Log the discussion using the absolute path: `<project-root>/<review-name>_review/log.sh "Claude" "<summary of decisions>"` — never use a relative `./log.sh` path, it will fail the permission check
   - Update `inventory.md` to mark the item complete
   - Present the section progress table with updated status
   - Move to the next item

### Between Sections

After completing all items in a section:
- Summarize what was decided (artifacts created, items marked clean)
- Show the full inventory with section-level completion status
- Move to the next section

### Artifact Creation

When the user gives an instruction requiring an artifact:
- Create it immediately, in the same turn
- Use the format negotiated in Phase 0
- If the format is GitHub issues, invoke the `/issue` skill via the Skill tool — do NOT call `mcp__github__create_issue` directly
- Number artifacts sequentially (e.g. FINDING-001, SPEC-001)
- Each artifact should be self-contained — readable without context from the conversation
- Announce: "Created [ARTIFACT-NNN]: [title]"

---

## Phase 2: Wrap-up

After all sections are reviewed:

1. **Produce a summary document** at `<review-name>_review/summary.md`:
   - Total items reviewed
   - Total artifacts produced (with titles and links/paths)
   - Cross-cutting themes or patterns noticed
   - Open questions or deferred items

2. **Open the report in plannotator** for review:
   ```
   /plannotator-annotate <review-name>_review/report.md
   ```
   For each annotation: address it, update `report.md`, and log the decision to the discussion log using the absolute path to `log.sh`.

3. **Log the final summary** to the discussion log

4. **Commit** all review artifacts:
   ```bash
   git add <review-name>_review/
   git commit -m "Add <review-name> review artifacts"
   ```

4. **Ask** if there is anything to revisit or if the review is complete

5. **Offer brainstorm handoff** — if the interview surfaced problems, gaps, or opportunities worth designing a solution for:

   > "We've covered the problem space. Want to move into designing a solution? I can hand off to `/brainstorm` with everything we've discussed."

   If the user says yes, invoke the `brainstorm` skill with a summary referencing the review artifacts.

---

## Guarding the Interview

The interview's primary job is knowledge transfer — getting what the user knows into a structured, shared understanding. Protect this:

- **If the user starts proposing solutions mid-interview**, redirect: "Hold that thought — I want to make sure I understand the full picture before we start designing. Is there anything else about [current topic] I should know?"
- **If the user says "let's just fix it"**, check: "Before we jump to solutions — is there more context I need? Constraints, history? The better I understand the problem, the better the design will be."
- **Do not resist indefinitely.** If the user pushes back, respect it and offer the brainstorm handoff.

---

## Interaction Rules

1. **One chunk at a time.** Never present a wall of information.
2. **Wait for a response before advancing.** Do not move to the next chunk, item, or section until the user has responded.
3. **Progress is always visible.** Every chunk presentation shows: which section, which item, which chunk, how many remain.
4. **Artifacts are created immediately.** When the user gives an instruction, create the artifact in the same turn.
5. **The discussion log is append-only.** Use `log.sh` to record key decisions. The log survives `/clear` and session restarts.
6. **Ask, do not assume.** If an instruction is ambiguous, ask one clarifying question before creating an artifact.
7. **Be opinionated.** Flag things that look wrong, inconsistent, or surprising. Say "this feels like it might belong elsewhere" or "this overlaps with X."
8. **Adapt to the user's pace.** Rapid-fire approvals → pick up pace. Deep dives → slow down.

