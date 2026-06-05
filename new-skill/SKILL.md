---
name: new-skill
description: This skill should be used when the user wants to create a new Claude Code skill and wire it up for global use. Handles writing the SKILL.md with proper frontmatter, plannotator review, test scenario generation, git commit, push, and symlinking. Use as /new-skill <name> or /new-skill <name> private for private skills.
user-invocable: true
effort: medium
---

Create a new Claude Code skill and wire it up for global use.

The user will invoke this as `/new-skill <name>` or `/new-skill` (and you'll ask for the name).

## Steps

1. **Get the skill name and visibility** — use the argument provided, or ask for the name if not given. If the user explicitly says "private" (in the invocation or context), the skill goes in the private repo. Otherwise it is public. Skill repo paths are defined in CLAUDE.md Skills Workflow section.

2. **Get the skill description** — ask the user: "What should `/<name>` do?" Let them describe it in plain language, including trigger conditions, key steps, and expected outputs. Ask any clarifying questions up front — the goal is to capture enough detail to write the full skill in one pass without further back-and-forth.

3. **Spawn parallel research while drafting** — once you have the description, do both at the same time:

   a. **Explore agent** (background, `subagent_type=Explore`): Survey all `SKILL.md` files in both skill repos (paths from CLAUDE.md) and:
      - Identify any skills whose behavior overlaps with or could be invoked as a substep of the new skill. Return: skill name, which substep it covers, whether to call it via the Skill tool.
      - Identify any existing specialized distiller/reviewer *agents* (agent type definitions, e.g. `resume-analyst`) that could be reused by the new skill.
      - Assess whether the new skill's description warrants a new companion distiller agent (reads heavy sources, returns distilled output) — report yes/no and why.

   b. **Draft the SKILL.md body** inline while the agent runs. Do not wait for it. Write the best first-pass you can without the reuse findings.

4. **Incorporate reuse** — when the Explore agent returns, revise the draft to invoke reusable skills via the Skill tool and reuse existing agents rather than reproducing their behavior inline. Prefer delegation over duplication.

4a. **Apply the agent-forward pattern** — Read `~/.claude/AGENT-FORWARD.md`. Run its applicability test against the skill's description:
   - Reads ≥1 large source OR ≥3 files? → apply P1, P2, P6 (front-loading analyst, paths not contents).
   - Fans out to parallel agents? → apply P3, P4 (compress shared context once; batch 3–5 items per agent).
   - Multi-phase or long-running? → apply P5 (staging file + `continue` mode).

   For each applicable principle, revise the draft to embody it — design the front-loading analyst, the shared-context block, batching, or staging file as concrete steps. If no principles apply (thin/single-shot skill), add a one-line note in the draft: "This is a single-pass skill; the agent-forward pattern does not apply."

5. **Write the skill file** — create `<repo>/<name>/SKILL.md`. The file MUST start with YAML frontmatter:

   ```yaml
   ---
   name: <skill-name>
   description: <third-person trigger description — starts with "This skill should be used when...">
   user-invocable: true
   ---
   ```

   Then the body: imperative tone, step-by-step where appropriate, no unnecessary preamble. Do not hardcode user-specific absolute paths — derive them dynamically (git commands, CLAUDE.md references, symlink resolution) or note them as user-configurable. The body must reflect the agent-forward decisions from Step 4a: if a front-loading analyst, batching, or staging file was prescribed, those must appear as named steps.

6. **Generate test scenarios** — invoke the `/skill-tdd generate <name>` skill via the Skill tool to create `<repo>/<name>/tests/scenarios.md`. For a new skill, generated scenarios will cover the main happy path, the primary error/bad-input case, and any edge cases implied by the SKILL.md steps.

7. **Review with plannotator** — invoke the `plannotator-annotate` skill to open the SKILL.md for review. Remind the user they must interact with at least one element (e.g. 👍 on the title) before closing — just closing the tab will hang the process. Incorporate any feedback before proceeding.

8. **Commit and symlink** — from the appropriate skill repo:

   ```bash
   git -C <repo-path> add <name>/
   git -C <repo-path> commit -m "Add <name> skill"
   git -C <repo-path> push
   ln -sf <repo-path>/<name> ~/.claude/skills/<name>
   ```

9. **Update memory** — add the new skill in two places:
   - **CLAUDE.md Available skills section** — add a one-line entry: `` - `/<name>` — <one-line description> ``
   - **MEMORY.md** — add an entry under "Skills" pointing to the skill
   If the skill covers something previously tracked as prose in memory, replace that entry with the skill reference.

10. **Remind the user** that the skill won't be available until the next session restart.
