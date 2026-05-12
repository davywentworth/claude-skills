---
name: verify-config
description: Use after any config change (settings, permissions, hooks, env vars) to confirm it actually took effect. Reports PASS/FAIL with evidence — never assumes success.
---

Given a recent config change, verify it took effect:

1. Read the relevant settings file and confirm the expected key/value is present.
2. Check if a reload or restart is required for the change to apply (e.g. Claude Code session restart, shell reload, service restart).
3. Test the change with the smallest possible action that would exercise it (e.g. run the permitted command, read the permissioned path, trigger the hook).
4. Report PASS/FAIL with concrete evidence — quote the actual output or file contents. Never assume success.
