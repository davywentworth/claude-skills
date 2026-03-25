Create a new Claude Code skill and wire it up for global use.

The user will invoke this as `/new-skill <name>` or `/new-skill` (and you'll ask for the name).

## Steps

1. **Get the skill name** — use the argument provided, or ask the user for it if not given.

2. **Get the skill description** — ask the user: "What should `/name` do?" Let them describe it in plain language. Ask any clarifying questions needed to write a complete, unambiguous skill.

3. **Check existing skills for reuse** — read all files in `/Users/davy/dev/claude-skills/` and identify any whose behavior overlaps with substeps of the new skill. Where a substep is already handled by an existing skill, the new skill should invoke that skill via the Skill tool rather than reproducing the behavior inline. Note which skills will be reused before drafting.

4. **Write the skill** — create `/Users/davy/dev/claude-skills/<name>.md` with clear instructions. Follow the same style as existing skills in that directory: imperative tone, step-by-step where appropriate, no unnecessary preamble. Incorporate invocations of existing skills identified in step 3.

5. **Review with plannotator** — invoke the `plannotator-annotate` skill to open the file for review. Remind the user they must interact with at least one element (e.g. 👍 on the title) before closing — just closing the tab will hang the process. Incorporate any feedback before proceeding.

6. **Commit and symlink**
   ```bash
   cd /Users/davy/dev/claude-skills
   git add <name>.md
   git commit -m "Add <name> skill"
   git push
   ln -s /Users/davy/dev/claude-skills/<name>.md ~/.claude/commands/<name>.md
   ```

7. **Update memory** — add a reference to the new skill in the relevant memory file(s) if it covers something previously tracked in memory, or note it in MEMORY.md under Claude Skills.

8. **Remind the user** that the skill won't be available until the next session restart.
