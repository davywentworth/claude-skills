---
name: verification-before-completion
description: Use when about to claim work is complete, before committing or creating PRs — requires running verification commands and confirming output. Evidence before assertions, always.
---

> Adapted from: https://github.com/anutron/ai/blob/main/skills/verification-before-completion/SKILL.md
> Upstream SHA: a5bd97a37cf0f57179e30e5415243a1c4dd40d05

# Verification Before Completion

## Overview

Claiming work is complete without verification is dishonesty, not efficiency.

**Core principle:** Evidence before claims, always.

**Violating the letter of this rule is violating the spirit of this rule.**

## The Iron Law

```
NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE
```

If you haven't run the verification command in this message, you cannot claim it passes.

## The Gate Function

```
BEFORE claiming any status or expressing satisfaction:

1. IDENTIFY: What command proves this claim?
2. RUN: Execute the FULL command (fresh, complete)
3. READ: Full output, check exit code, count failures
4. VERIFY: Does output confirm the claim?
   - If NO: State actual status with evidence
   - If YES: State claim WITH evidence
5. ONLY THEN: Make the claim

Skip any step = lying, not verifying
```

## Common Failures

| Claim | Requires | Not Sufficient |
|-------|----------|----------------|
| Tests pass | `npm test` output: 0 failures | Previous run, "should pass" |
| Linter clean | `npm run lint` output: 0 errors | Partial check, extrapolation |
| Build succeeds | Build command: exit 0 | Linter passing, looks good |
| Bug fixed | Test original symptom: passes | Code changed, assumed fixed |
| Agent completed | VCS diff shows changes | Agent reports "success" |
| Requirements met | Line-by-line checklist | Tests passing |

## Red Flags — STOP

- Using "should", "probably", "seems to"
- Expressing satisfaction before verification ("Great!", "Perfect!", "Done!")
- About to commit/push/PR without verification
- Trusting agent success reports without checking the diff
- Relying on partial verification
- **Any wording implying success without having run verification**

## Rationalization Prevention

| Excuse | Reality |
|--------|---------|
| "Should work now" | RUN the verification |
| "I'm confident" | Confidence ≠ evidence |
| "Just this once" | No exceptions |
| "Linter passed" | Linter ≠ test suite |
| "Agent said success" | Verify independently |
| "Partial check is enough" | Partial proves nothing |

## Key Patterns

**Tests (Node/TypeScript projects):**
```
✅ cd backend && npm test  →  see: X passing, 0 failing  →  "All tests pass"
✅ cd frontend && npm test  →  see: X passing, 0 failing  →  "All tests pass"
❌ "Should pass now" / "Looks correct"
```

**Lint:**
```
✅ npm run lint  →  see: 0 errors  →  "Lint clean"
❌ "I think it's fine"
```

**Bug fixed:**
```
✅ Write test reproducing bug → Run (FAILS) → Fix → Run (PASSES) → "Bug fixed"
❌ "I changed the code, it should be fixed"
```

**Requirements met:**
```
✅ Re-read plan/issue → Create checklist → Verify each item → Report gaps or completion
❌ "Tests pass, I'm done"
```

**Agent delegation:**
```
✅ Agent reports success → Check git diff → Verify changes match intent → Report actual state
❌ Trust agent report at face value
```

## When To Apply

**Always before:**
- Any success or completion claim
- Any expression of satisfaction with work state
- Committing, PR creation, task completion
- Moving to the next task
- Delegating to agents and reporting their results

## The Bottom Line

Run the command. Read the output. Then claim the result.

No shortcuts. No exceptions.
