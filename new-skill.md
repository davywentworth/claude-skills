Create a new Claude Code skill and wire it up for global use.

The user will invoke this as `/new-skill <name>` or `/new-skill` (and you'll ask for the name).

## Steps

1. **Get the skill name** — use the argument provided, or ask the user for it if not given.

2. **Get the skill description** — ask the user: "What should `/name` do?" Let them describe it in plain language. Ask any clarifying questions needed to write a complete, unambiguous skill.

3. **Write the skill** — create `/Users/davy/dev/claude-skills/<name>.md` with clear instructions. Follow the same style as existing skills in that directory: imperative tone, step-by-step where appropriate, no unnecessary preamble.

4. **Show it to the user** — print the skill content and ask if they want any changes before proceeding.

5. **Commit and symlink**
   ```bash
   cd /Users/davy/dev/claude-skills
   git add <name>.md
   git commit -m "Add <name> skill"
   git push
   ln -s /Users/davy/dev/claude-skills/<name>.md ~/.claude/commands/<name>.md
   ```

6. **Update memory** — add a reference to the new skill in the relevant memory file(s) if it covers something previously tracked in memory, or note it in MEMORY.md under Claude Skills.

7. **Remind the user** that the skill won't be available until the next session restart.
