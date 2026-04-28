---
name: guard
description: Use before any git commit to check for secrets, security antipatterns, and test breakage. Fast binary pass/fail pre-commit gate.
---

> Adapted from: https://github.com/anutron/ai/blob/main/skills/guard/SKILL.md
> Upstream SHA: 609c1132276ac77e1f6043c72f062ccc7f367195

# Pre-commit Guard

Fast safety check before committing. Scans for secrets, security antipatterns, test breakage, and lint issues. Binary pass/fail output.

## Arguments

- `$ARGUMENTS` - Optional: `--strict` for zero-tolerance mode (warnings also fail)

## Context

- Staged files: !`git diff --cached --name-only`
- Unstaged changes: !`git diff --name-only`
- Project type: !`find . -maxdepth 2 \( -name go.mod -o -name package.json -o -name pyproject.toml -o -name Cargo.toml \) 2>/dev/null | head -5`

## Instructions

Run these checks against all staged files (or all changed files if nothing is staged). Be fast — this is a pre-commit gate, not a full review.

### Check 0: Gitignore Violations

Scan staged/changed files for common files that should not be committed:

- `.env`, `.env.*` files (environment/secrets)
- `node_modules/` contents
- Build artifacts (`dist/`, `build/`, `*.o`, `*.pyc`, `__pycache__/`)
- IDE files (`.idea/`, `.vscode/settings.json`)
- OS files (`.DS_Store`, `Thumbs.db`)
- Log files (`*.log`)
- Credential files (`credentials.json`, `*.pem`, `*.key`)
- Coverage directories (`coverage/`)

**Report:** WARNING if any found (FAIL in --strict mode).

### Check 1: Secrets Detection

Scan staged/changed files for:

- API keys, tokens, passwords (patterns: `password\s*=`, `api_key`, `secret`, `token\s*=`, `Bearer `)
- AWS credentials (`AKIA`, `aws_secret_access_key`)
- Private keys (`-----BEGIN.*PRIVATE KEY-----`)
- Connection strings with embedded credentials
- `.env` files being committed
- Hardcoded URLs with credentials (`https://user:pass@`)
- `ANTHROPIC_API_KEY` or other AI provider keys hardcoded in source

**Report:** FAIL if any found, with file and line number.

### Check 2: Security Antipatterns

Scan for common dangerous patterns:

- `eval()` or `exec()` with variable input
- SQL string concatenation (instead of parameterized queries)
- `dangerouslySetInnerHTML` or equivalent
- Disabled SSL/TLS verification
- Overly permissive CORS (`Access-Control-Allow-Origin: *`)
- `--no-verify` or security bypass flags in code

**Report:** FAIL if any found.

### Check 3: Test Breakage

Run a quick, targeted test check:

```
IF package.json exists in backend/ or frontend/:
  Identify which packages contain changed files
  For each affected package: cd <package> && npm test -- --run 2>&1 | tail -20
  Report: PASS or FAIL with failure summary
ELSE IF other test framework detected:
  Run equivalent targeted test command
ELSE:
  Report: SKIP (no test framework detected)
```

Keep this fast — targeted tests for changed packages only, not the full suite.

### Check 4: Lint Check

```
IF package.json exists in affected package(s):
  cd <package> && npm run lint 2>&1 | tail -20
IF go.mod exists: go vet ./...
IF pyproject.toml exists: ruff check {changed files} (if ruff configured)
ELSE: SKIP
```

**Report:** FAIL if lint errors found.

### Output

```
IF --strict mode:
  FAIL if ANY check has warnings or failures
ELSE:
  FAIL if ANY check has failures (warnings are noted but pass)
```

Format:

```markdown
## Guard Check

| Check | Status | Details |
|-------|--------|---------|
| Gitignore | PASS/WARN | {brief or "Clean"} |
| Secrets | PASS/FAIL | {brief or "Clean"} |
| Security | PASS/FAIL | {brief or "Clean"} |
| Tests | PASS/FAIL/SKIP | {brief or "All passing"} |
| Lint | PASS/FAIL/SKIP | {brief or "Clean"} |

**Result: PASS / FAIL**

{If FAIL: list each issue with file and line number}
{If PASS: "Safe to commit."}
```
