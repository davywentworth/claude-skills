---
name: verify-config
description: This skill should be used after any config change (settings, permissions, hooks, env vars) to confirm it actually took effect. Reports PASS/FAIL with concrete evidence — never assumes success.
effort: low
model: haiku
---

Given a recent config change, verify it took effect:

1. Read the relevant settings file and confirm the expected key/value is present.
2. Determine if a reload or restart is needed:
   - **Claude Code settings** (`settings.json`, `settings.local.json`, `CLAUDE.md`): require a session restart. If still in the same session, note the change won't apply until restart.
   - **Shell config** (`.bashrc`, `.zshrc`, `.profile`): require a shell reload (`source ~/.bashrc`) or new terminal.
   - **Process-based config** (env vars, service configs): require restarting the process.
   - **File-read-on-use config** (read at runtime, not startup): takes effect immediately once the file is written.
3. Test the change with the smallest possible action that would exercise it (e.g. run the permitted command, read the permissioned path, trigger the hook).
4. Report PASS/FAIL with concrete evidence — quote the actual output or file contents. Never assume success.
