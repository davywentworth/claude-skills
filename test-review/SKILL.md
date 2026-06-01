---
name: test-review
description: This skill should be used when the user wants an antagonistic test quality review. Checks test files against 4 criteria — minimal mocking, well-named, clearly written, antagonistic coverage — and fixes issues in a loop until clean.
user-invocable: true
effort: high
---

Antagonistic test quality review. Checks test files against 4 criteria: minimal mocking, well-named, clearly written, and antagonistic coverage.

## Steps

1. **Identify test files** — find all `*.test.ts`, `*.test.tsx`, `*.spec.ts`, `*.spec.tsx` files that have changes (`git diff HEAD --name-only`). If no changed test files exist, use any test files the user specified. If none, ask the user which files to review.

2. **Spawn parallel agents** — for each test file, spawn a separate Agent (subagent_type: test-reviewer) IN PARALLEL. Each agent receives:
   - The full content of the test file
   - The full content of the file(s) being tested

3. **Print all findings** in full, grouped by file. Do not summarize or omit anything — the user reads the raw output.

4. **Fix real issues** — fix all high severity issues without asking for confirmation. For medium severity issues, present them to the user and fix upon confirmation. For low severity issues, describe them and ask whether to fix.

5. **Re-run** — after fixing, repeat from step 2 on the modified files. Loop until all agents report no high or medium issues.

6. **Run tests** — once the review is clean, detect the test runner from the project structure (`package.json` → npm test, `Makefile` → make test, `pyproject.toml` → pytest, `go.mod` → go test ./...) and run the appropriate command for each package containing reviewed files to confirm all tests pass.
