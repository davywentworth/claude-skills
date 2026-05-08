# Job Scan

Scan for new qualifying job postings across Dice, Indeed, DuckDuckGo's careers page, and B-corp / mission-aligned tech companies. Deduplicates against existing findings, adds qualifying roles to `~/research/job-search/README.md`, and suggests `/tailor-resume` for top matches.

## Steps

### 1. Load context

Read both files in parallel:
- `~/research/job-search/rules.md` — disqualifiers, salary floor, fit criteria, search queries
- `~/research/job-search/README.md` — existing Promising Roles, Applied, Disqualified tables, and Search Log

Build a dedup set: all role titles + companies already in Promising Roles, Applied, or Disqualified. Any match = skip silently.

### 2. Check DuckDuckGo careers page

Fetch `https://api.ashbyhq.com/posting-api/job-board/duck-duck-go?includeCompensation=true`.

Look for any role with "frontend", "React", "TypeScript", or "JavaScript" in the title or description. Also check whether the previously closed `Senior Frontend Engineer, React/TypeScript` (Ashby ID `5d3e230c-ecf4-407a-a3a5-232185ce8319`) has been re-posted.

### 3. Run Dice searches

Use `mcp__dice__search_jobs` with `workplace_types=Remote` and `employment_types=FULLTIME` for each query:

1. `lead software engineer React TypeScript remote`
2. `senior software engineer React TypeScript remote`
3. `principal software engineer React TypeScript remote`
4. `staff software engineer React TypeScript remote`
5. `senior frontend engineer React TypeScript remote`

Apply all disqualifiers from rules.md. Skip anything in the dedup set.

### 4. Run Indeed searches

Use `mcp__claude_ai_Indeed__search_jobs` for each query (US, FT, remote):

1. `senior software engineer React TypeScript remote`
2. `lead software engineer React TypeScript full stack remote`
3. `senior frontend engineer React TypeScript remote`
4. `principal software engineer React TypeScript remote`
5. `staff frontend engineer React TypeScript remote`

Apply same disqualifier filter and dedup check.

### 5. B-corp and mission-aligned search

Run WebSearch queries:

1. `"B corp" certified tech company senior React TypeScript engineer remote hiring 2026`
2. `privacy tech company senior frontend engineer React TypeScript remote 2026`
3. `civic tech "public benefit" senior software engineer React TypeScript remote 2026`

For each promising company found, WebFetch their careers page to confirm open React/TS roles. Apply all disqualifier rules. DuckDuckGo, Nava PBC, and Ad Hoc are already tracked — skip unless new roles appear.

For any new company with a confirmed open role, invoke `/research` to build an employer profile (Glassdoor, benefits, stability, what employees say) before adding it to the README. Pass the company name as the query.

### 6. Verify promising findings

For every new candidate that passed initial filters: WebFetch the company's own careers page to confirm the role is still open (Verification Rule from rules.md). If not confirmed open on the company's own site or ATS, do not add it.

### 7. Update README

For each confirmed new qualifying role:

**Promising Roles table** — insert a new row at the appropriate rank:
```
| <rank> | <date> | [Title](url) | Company | $salary | Track | **Fit** | **Apply** — <brief reason> |
```

**All Listings section** — append a full listing block:
```
### [Title](url) — Company
**Date found:** YYYY-MM-DD | **Salary:** $ | **Track:** | **Stack fit:**
**Source:** | **Job ID:** <id>

> <1–2 sentence description>

**Why apply:** <stack match, salary, remote, experience fit, standout factors>

**Red flags:** <any concerns>
```

**Search Log** — append one row per search run:
```
| <date> | <platform> | <query> | <result summary> |
```

For any new company that needed an employer profile (Step 5), invoke `/kb-save` with the topic `"<Company> employer research"` to write the profile into the Company Notes section of the job-search README.

### 8. Report

Output a summary:
- New qualifying roles found (or "No new qualifying roles found")
- For each: title, company, salary, why it qualifies
- Whether DuckDuckGo has any new frontend openings
- Any notable near-misses or companies worth monitoring

For each **Perfect** or **Strong** fit role found, end with:
> Run `/tailor-resume <url>` to produce a gap analysis and resume rewrites for this role.
