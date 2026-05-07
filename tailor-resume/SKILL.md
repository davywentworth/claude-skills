---
name: tailor-resume
description: Gap analysis + resume bullet rewrites for a specific job req, sourcing from the about-my-career KB. Checks disqualifiers first, then produces a gap table, targeted rewrites, and a tailored summary line.
---

# Tailor Resume to a Job Req

Takes a job URL or pasted JD and produces a gap analysis plus concrete resume rewrites using the career background KB as source material. Output is conversational — the user picks what to apply.

**Never navigate to or interact with application forms.** Research and prep only — the user submits manually. AI traps on applications can disqualify candidates.

## Arguments

- `<url-or-jd>` — job posting URL or raw JD text (required)

## Steps

### 1. Check disqualifiers first

Read `~/research/job-search/rules.md`. Before doing any other work, scan the JD against the Disqualify section. If a hard disqualifier is present (domain objection, on-site, under $150k, under 7 yrs required, startup), flag it immediately and stop:

> "This role fails a hard disqualifier: [reason]. Not proceeding with gap analysis."

### 2. Fetch the JD

If input is a URL: WebFetch it for the full job description — title, requirements, responsibilities, salary, remote status, stack, years of experience required.

If input is pasted text: use as-is.

### 3. Read the career background

Read both sources in parallel:

**`~/research/about-my-career/README.md`** — narrative source of truth:
- Per-role narratives and accomplishments
- Honest skills gradient (strongest → weakest)
- Application guidance by role type
- Narrative framings (reinvention story, values pivot, design-engineering positioning, AI workflow)

**Latest full resume** — find the most recently modified PDF matching `Davy Wentworth - Senior Engineer_Lead Resume - Full.pdf` in `~/Downloads/`. Read it for the current bullet text, section structure, and exact wording. Rewrites in Step 6 should reference the actual current bullets, not reconstructed versions.

### 4. Check existing company research

Read `~/research/job-search/README.md` — scan the Company Notes section for an existing entry on this employer. If none exists and the company is not well-known, invoke `/research` to build an employer profile before proceeding.

### 5. Produce the gap analysis table

| JD Requirement | Resume Signal | Assessment |
|---|---|---|
| [requirement] | [what the background doc has] | Strong / Partial / Gap |

Cover all material requirements from the JD. For each row:
- **Strong** — directly evidenced in the career background doc
- **Partial** — present but undersold, not framed correctly, or requires rewriting to land
- **Gap** — not evidenced; note whether it's a hard gap (missing skill) or a soft gap (framing/narrative)

### 6. Produce targeted rewrites

For each **Gap** or **Partial** row, write a specific improved bullet or phrase using material from the career background doc. Group rewrites by resume section:

**Take2 bullets** — rewrites for the current role section  
**Thanx bullets** — rewrites pulling from the right phase (Lead / Senior / Early)  
**Skills section** — any changes to how skills are listed or ordered  
**Summary line** — one rewritten summary sentence optimized for this role

Format each rewrite as:
> **Current:** [existing bullet or "missing"]  
> **Rewrite:** [improved version]  
> **Why:** [what gap this addresses]

### 7. Narrative gap notes

Separate from the technical gap table, flag any framing or narrative adjustments:
- Which career narrative to lead with for this role type (use the Application Guidance section of about-my-career/README.md)
- Any values alignment or mission framing the cover letter should address
- Any background to downplay or reframe (e.g. full-stack → frontend lean for pure frontend roles; Thanx loyalty/marketing context for privacy-focused roles)

### 8. Summarize

End with:
- Overall fit verdict: **Strong / Acceptable / Stretch**
- Top 2–3 things to do before applying (specific bullets to update, cover letter angle, anything to verify)
- Whether to add this role to the job-search README (and at what rank)
