End-of-session skill improvement and suggestions. Run this at the end of a working session.

## Steps

1. **Audit skills used this session**
   - Review the full conversation context to identify which skills were invoked (e.g. `/review`, `/issue`, `/issue plan`).
   - For each skill, note: did it work smoothly? Were there corrections, workarounds, or friction? Did the user adjust the output or push back on anything?

2. **Gather feedback**
   - Look for explicit feedback ("don't do X", "it should also Y") and implicit feedback (user edited skill output, corrected behavior, or repeated a request because the first result missed the mark).

3. **Propose skill updates**
   - For each skill with actionable feedback, read the current skill file fresh from `/Users/davy/dev/claude-skills/` — do not rely on the version loaded into context, which may be stale if the skill was updated mid-session.
   - Also audit skills *created* this session — were there corrections or additions needed before they were finalized?
   - Draft the proposed changes and present them clearly to the user before making any edits.
   - Note: skills created mid-session won't be recognized by Claude Code until the next session restart. This is expected behavior.

4. **Audit permissions**
   - Review the conversation for any tool calls the user was prompted to approve (look for rejected or interrupted tool calls, and commands where the user had to explicitly allow).
   - For each one, assess whether it's safe to auto-approve globally (e.g. read-only `gh` commands, posting comments) vs. commands that warrant case-by-case approval (e.g. destructive operations, pushes).
   - Read `~/.claude/settings.json` and propose adding safe commands to the `permissions.allow` array using the `Bash(<prefix>:*)` pattern.
   - Include these proposals in the approval step alongside skill updates.

5. **Suggest new skills**
   - Identify patterns in the session that were handled ad-hoc but would benefit from a reusable skill.
   - Present suggestions to the user: skill name, one-line description, and why it would be useful based on what happened this session.

6. **Wait for approval**
   - Present all proposed skill updates, new skill suggestions, and permission additions together.
   - Wait for the user to approve or reject each one before proceeding.

7. **Apply approved changes**
   - Make all approved skill edits and create any approved new skills (write to `/Users/davy/dev/claude-skills/<name>.md`, symlink to `~/.claude/commands/<name>.md`).
   - Apply any approved permission additions to `~/.claude/settings.json`.
   - Commit and push: `cd /Users/davy/dev/claude-skills && git add -A && git commit -m "Session improvements: <summary>" && git push`

8. **Update memory**
   - For any skills created or meaningfully updated, update the relevant memory files under `~/.claude/projects/`.
   - If a skill now covers something that was previously tracked in memory, trim the redundant memory entry and add a pointer to the skill instead.
   - If new patterns or preferences emerged this session (user corrections, workflow changes, things to always/never do), save them as new feedback memories or update existing ones.
   - Update `MEMORY.md` index to reflect any new or changed memory files.
