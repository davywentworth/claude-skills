---
name: new-skill
description: This skill should be used when the user wants to create a new Claude Code skill and wire it up for global use. Handles writing the SKILL.md with proper frontmatter, plannotator review, test scenario generation, git commit, push, and symlinking. Use as /new-skill <name> or /new-skill <name> private for private skills.
user-invocable: true
---

Create a new Claude Code skill and wire it up for global use.

The user will invoke this as `/new-skill <name>` or `/new-skill` (and you'll ask for the name).

## Steps

1. **Get the skill name** — use the argument provided, or ask the user for it if not given.

2. **Determine visibility** — if the user explicitly says "private" (in the invocation or context), the skill goes in the private repo. Otherwise it is public. Skill repo paths are defined in CLAUDE.md Skills Workflow section.

3. **Get the skill description** — ask the user: "What should `/name` do?" Let them describe it in plain language. Ask any clarifying questions needed to write a complete, unambiguous skill.

4. **Check existing skills for reuse** — read all `SKILL.md` files in both skill repos (paths from CLAUDE.md Skills Workflow section) and identify any whose behavior overlaps with substeps of the new skill. Where a substep is already handled by an existing skill, the new skill should invoke that skill via the Skill tool rather than reproducing the behavior inline. Note which skills will be reused before drafting.

5. **Write the skill** — create `<repo>/<name>/SKILL.md`. The file MUST start with YAML frontmatter:

   ```yaml
   ---
   name: <skill-name>
   description: <third-person trigger description — starts with "This skill should be used when...">
   user-invocable: true
   ---
   ```

   Then the body: imperative tone, step-by-step where appropriate, no unnecessary preamble. Incorporate invocations of existing skills identified in step 4. Do not hardcode user-specific absolute paths — derive them dynamically (git commands, CLAUDE.md references, symlink resolution) or note them as user-configurable.

6. **Generate test scenarios** — invoke the `/skill-tdd generate <name>` skill via the Skill tool to create `<repo>/<name>/tests/scenarios.md`. For a new skill, generated scenarios will cover the main happy path, the primary error/bad-input case, and any edge cases implied by the SKILL.md steps.

7. **Review with plannotator** — invoke the `plannotator-annotate` skill to open the SKILL.md for review. Remind the user they must interact with at least one element (e.g. 👍 on the title) before closing — just closing the tab will hang the process. Incorporate any feedback before proceeding.

8. **Commit and symlink** — from the appropriate skill repo:

   ```bash
   git add <name>/
   git commit -m "Add <name> skill"
   git push
   ln -sf <repo-path>/<name> ~/.claude/commands/<name>
   ```

9. **Update memory** — add the new skill in two places:
   - **CLAUDE.md Available skills section** — add a one-line entry: `` - `/<name>` — <one-line description> ``
   - **MEMORY.md** — add an entry under "Skills" pointing to the skill
   If the skill covers something previously tracked as prose in memory, replace that entry with the skill reference.

10. **Remind the user** that the skill won't be available until the next session restart.
