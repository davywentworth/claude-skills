Antagonistic test quality review. Checks test files against 4 criteria: minimal mocking, well-named, clearly written, and antagonistic coverage.

## Steps

1. **Identify test files** — find all `*.test.ts`, `*.test.tsx`, `*.spec.ts`, `*.spec.tsx` files that have changes (`git diff HEAD --name-only`). If no changed test files exist, use any test files the user specified. If none, ask the user which files to review.

2. **Spawn parallel agents** — for each test file, spawn a separate Agent (subagent_type: general-purpose) IN PARALLEL. Each agent receives:
   - The full content of the test file
   - The full content of the file(s) being tested
   - This instruction: "You are an antagonistic test reviewer. Review this test file against these 4 criteria:
     1. **Minimal mocking** — is infrastructure (DB, filesystem) mocked when it shouldn't be? Is a real implementation used where possible?
     2. **Well named** — do test names clearly describe the behavior being tested? Does reading the test suite feel like reading living documentation?
     3. **Clearly written** — can a reader quickly understand what each test does without deep context?
     4. **Antagonistic coverage** — are edge cases, error paths, and failure modes covered? Are there gaps that could hide real bugs?
     Be harsh but accurate — only flag real issues, not style nits covered by Prettier/ESLint. For each issue: state which criterion it violates, severity (high/medium/low), line number, explanation, and suggested fix. If the file fully passes a criterion, say so briefly."

3. **Print all findings** in full, grouped by file. Do not summarize or omit anything — the user reads the raw output.

4. **Fix real issues** — fix all high and medium severity issues without asking for confirmation. For low severity issues, describe them and ask whether to fix.

5. **Re-run** — after fixing, repeat from step 2 on the modified files. Loop until all agents report no high or medium issues.

6. **Run tests** — once the review is clean, run `cd backend && npm test` and/or `cd frontend && npm test` (whichever packages contain the reviewed files) to confirm all tests pass.
