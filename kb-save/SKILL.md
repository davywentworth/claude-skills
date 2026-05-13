---
name: kb-save
description: This skill should be used after research sessions, employer research, or any conversation that produces knowledge worth persisting to ~/research/. Handles slug derivation, new-vs-expansion detection, frontmatter, manifest update, and cross-linking.
---

# Save to Research Knowledge Base

Stores conversation-generated content in `~/research/` using the same structure as the `/research` skill. The content source is this conversation, not web agents.

## Arguments

- `<topic>` — what to save (e.g. "DuckDuckGo employer research", "about my career", "VA contractor landscape")
- `--section "<heading>"` — optional: only save a specific section of the conversation, not everything

If no argument is given, ask: "What topic should I save to the KB?"

## Steps

### 1. Determine the content

If `--section` was provided, focus only on the relevant portion of the conversation.

Otherwise, synthesize everything discussed this session about the topic into a coherent body of findings. Do not pad — only include what was actually established.

### 2. Derive the slug

Convert the topic to a slug: lowercase, spaces and special characters → hyphens, collapse repeated hyphens, strip trailing hyphens, max 40 chars.

Example: "DuckDuckGo employer research" → `duckduckgo-employer-research`

### 3. Check for existing or similar docs

Read `~/research/manifest.json`. Check two things:

**Exact slug match:** if `~/research/<slug>/README.md` exists → **expansion mode**.

**Semantic overlap:** scan `title`, `tags`, and `summary` of all entries. Consider a match if at least 2 tags overlap or there is a clear topical parent/child relationship. If a match is found, ask: "I found a related doc: `<title>` (`<slug>`). Should I add this content there, or create a new entry?"

If no match → **new mode**.

### 4. Check for placement guidance

Before writing, check whether this content belongs in an existing doc's section rather than as a standalone KB entry. Specifically:

- **Employer research for a job search company** → the full profile belongs in `~/research/job-search/company-research.md`; a one-line summary row belongs in the Company Notes table in `~/research/job-search/README.md`. Do not create a standalone entry.
- **Sub-topic of an existing doc** → expansion mode into the parent doc.
- **Genuinely standalone topic** → new entry.

### 5. Write the content

**New entry:**

Create `~/research/<slug>/` directory and write `README.md`:

```
---
title: <Human-readable title, Title Case>
date: <YYYY-MM-DD>
tags: [<tag1>, <tag2>, ...]
query: "<original topic as stated>"
summary: "<2–3 sentences written for Claude to scan in future sessions — core answer, key caveats, what makes this doc worth reading>"
related: []
---

## Overview
<Opinionated synthesis of what was established. Lead with highest-value findings. Note any open questions or pending items.>

## <Section 1>
...

## <Section N>
...

## Key Takeaways
- <bullet>
- <bullet>
```

Only include a `## References` section if web sources were cited in the conversation.

**Expansion into existing doc:**

Read the existing `~/research/<slug>/README.md`. Add new content:
- New sections go below existing ones
- New information within an existing section is appended as new paragraphs — never overwrite
- Rewrite the `## Overview` to reflect the complete current state (the one exception to no-overwrite)
- Update `date` in frontmatter to today

**Adding to job-search Company Notes:**

Find the `## Company Notes` section in `~/research/job-search/README.md`. Add a new `### <Company> — Employer Research` subsection using the established format (Glassdoor, role summary, benefits, what employees say, company health, domain, caution flags).

### 6. Cross-link

Read `~/research/manifest.json`. Find entries whose `tags` or `summary` overlap with this doc (≥2 matching tags or clear topical relationship). For each match:
- Add a `See also: [Title](../<slug>/)` inline reference in a relevant section
- Add the matched slug to the `related` frontmatter field
- Add a reciprocal link in the matched doc's most relevant section

### 7. Update manifest

Read `~/research/manifest.json` and update:
- **Existing entry:** update `date`, `tags`, `summary`, `related` in-place
- **New entry:** append a new object

Entry format:
```json
{
  "slug": "<slug>",
  "title": "<title>",
  "date": "<YYYY-MM-DD>",
  "tags": [...],
  "summary": "<summary>",
  "path": "<slug>/README.md",
  "related": [<slug1>, <slug2>]
}
```

Keep the array sorted by `date` descending. Write updated array back.

### 8. Confirm

Report what was written:
- Path of the file created or updated
- Whether it was a new entry or expansion
- Any cross-links added
- Manifest updated: yes/no
