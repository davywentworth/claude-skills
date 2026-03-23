End-of-session skill improvement and suggestions. Run this at the end of a working session.

## Steps

1. **Audit skills used this session**
   - Review the full conversation context to identify which skills were invoked (e.g. `/review`, `/issue`, `/issue plan`).
   - For each skill, note: did it work smoothly? Were there corrections, workarounds, or friction? Did the user adjust the output or push back on anything?

2. **Gather feedback**
   - Look for explicit feedback ("don't do X", "it should also Y") and implicit feedback (user edited skill output, corrected behavior, or repeated a request because the first result missed the mark).

3. **Update skills**
   - For each skill with actionable feedback, read the current skill file from `/Users/davy/dev/claude-skills/`.
   - Edit it to incorporate the improvements.
   - Commit the changes: `cd /Users/davy/dev/claude-skills && git add -p && git commit -m "Improve <skill-name> based on session feedback"`.

   - Also audit skills *created* this session — were there corrections or additions needed before they were finalized? Incorporate those learnings.
   - Note: skills created mid-session won't be recognized by Claude Code until the next session restart. This is expected behavior.

4. **Suggest new skills**
   - Identify patterns in the session that were handled ad-hoc but would benefit from a reusable skill.
   - Present suggestions to the user: skill name, one-line description, and why it would be useful based on what happened this session.
   - If the user approves any, create them following the standard workflow: write to `/Users/davy/dev/claude-skills/<name>.md`, commit, symlink to `~/.claude/commands/<name>.md`.

5. **Update memory**
   - For any skills created or meaningfully updated, update the relevant memory files under `~/.claude/projects/`.
   - If a skill now covers something that was previously tracked in memory, trim the redundant memory entry and add a pointer to the skill instead.
   - If new patterns or preferences emerged this session (user corrections, workflow changes, things to always/never do), save them as new feedback memories or update existing ones.
   - Update `MEMORY.md` index to reflect any new or changed memory files.

6. **Push**
   - `cd /Users/davy/dev/claude-skills && git push`
