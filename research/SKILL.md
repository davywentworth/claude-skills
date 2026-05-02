Conduct multi-agent web research and store results as a growing knowledge base at `~/research/`.

## Arguments

- `<query>` — what to research (required)
- `--no-research` — skip silently; used when called from planning skills that want to allow opt-out

## Steps

### 1. Check skip flag

If `--no-research` is in the arguments, stop immediately and return nothing.

### 2. Ensure knowledge base directory

If `~/research/` does not exist, create it. If `~/research/manifest.json` does not exist, create it with contents `[]`.

### 3. Derive slug

Convert the query to a slug: lowercase, spaces and special characters → hyphens, collapse repeated hyphens, strip trailing hyphens, max 40 chars.

### 4. Check for existing or similar docs

Read `~/research/manifest.json` (treat as `[]` if missing or unreadable). Check two things:

- **Exact slug match**: if `~/research/<slug>/README.md` exists, this is an expansion run — use the same slug.
- **Semantic overlap**: scan `title`, `tags`, and `summary` fields of all manifest entries for significant overlap with the query. Consider an entry a match if at least 2 tags overlap, or if there is a clear topical parent/child relationship. If multiple entries match, prefer the one with the most overlapping tags; if still tied, prefer the most recent. If a match is found, use that entry's slug for expansion.

Decide autonomously — do not prompt the user.

### 5. Spawn child orchestrator agent

Resolve the following values before spawning:
- **query**: the user's original query text
- **slug**: the slug resolved in Steps 3–4
- **mode**: `new` or `expansion`
- **date**: today's date (YYYY-MM-DD)

When constructing the child agent prompt below, substitute all `{{placeholder}}` tokens with these resolved values. Do not forward literal placeholder strings.

Spawn a **child agent** (`general-purpose` subagent_type) with these instructions:

---

You are the Research Orchestrator. Research the query below and produce a structured knowledge base document.

**Query:** (substituted query)
**Slug:** (substituted slug)
**Mode:** (new or expansion)
**Today's date:** (substituted date)

**If expansion mode:** read `~/research/<slug>/README.md` now — you will add to it, not replace it.

#### Step A — Decompose

Break the query into 0–6 subtopics based on complexity:
- **0 subtopics**: the topic is simple enough to handle yourself with direct WebSearch + WebFetch. Skip Step B. Write findings under a single `## Findings` section with inline `[N]` citations.
- **1–6 subtopics**: each will be handled by a dedicated grandchild agent in Step B.

#### Step B — Spawn grandchild agents in parallel

For each subtopic, resolve its name, then spawn one **grandchild agent** (`general-purpose`) per subtopic simultaneously. When constructing each grandchild prompt, substitute the actual subtopic name and query — do not pass literal placeholder text.

Each grandchild receives these instructions (with substituted values):

> You are researching the subtopic: **(subtopic name)**
> Broader context: **(query)**
>
> Steps:
> 1. Run 3–5 WebSearch queries covering different angles of this subtopic
> 2. WebFetch the top 2–3 most relevant results for full content
> 3. Return a structured findings block:
>    - A markdown section titled `## (subtopic name)`
>    - Findings written in clear prose with inline `[N]` citations (numbered starting from 1)
>    - A reference list: `[N]: <url> — <title>`

Collect all grandchild findings before proceeding.

#### Step C — Synthesize

Merge grandchild findings into the document body.

**For new docs**: assemble sections in order.

**For expansion**: integrate new findings into the existing document:
- Never overwrite existing content
- Add new subtopic sections below existing ones
- Expand existing sections in-place by appending new paragraphs
- **Citation renumbering**: find the highest reference number N already in the document. For every grandchild findings block, add N to each inline `[K]` citation (making it `[N+K]`) and update the reference list entries to match. This prevents collisions with existing references.
- Update `date` in frontmatter to today

#### Step D — Overview

**For new docs**: write a `## Overview` section to appear first in the body (after frontmatter).

**For expansion**: rewrite the `## Overview` section as a full replacement that synthesizes both old and new findings. The previous overview content is intentionally replaced — this is the one exception to the no-overwrite rule, because the overview should reflect the complete current state of the doc.

Either way, the Overview must:
- Summarize the most important findings across all subtopics
- Call out any conflicts or contradictions found between sources
- Lead with what is of highest value to a human reader — opinionated synthesis, not a dry list

#### Step E — Linking

Read `~/research/manifest.json`. Find entries whose `tags` or `summary` semantically overlap with this doc (same threshold as Step 4: at least 2 matching tags or a clear topical relationship). For each match:
- Add an inline cross-reference in a relevant section body (e.g. `See also: [Claude API](../claude-api/)`)
- Collect the matched slugs for the `related` frontmatter field

#### Step F — Write the document

Write to `~/research/<slug>/README.md` (create the `<slug>` directory if needed):

```
---
title: <human-readable title, title case>
date: <YYYY-MM-DD>
tags: [<tag1>, <tag2>, ...]
query: "<original query>"
summary: "<2–3 sentences written for Claude to scan in future sessions — cover the core answer, key caveats, and what makes this doc worth reading>"
related: [<slug1>, <slug2>]
---

## Overview
<opinionated synthesis: conflicts, key tensions, highest-value findings>

## <Subtopic 1>
...

## <Subtopic N>
...

## Key Takeaways
- <bullet>
- <bullet>

## References
[1]: <url> — <title>
[2]: <url> — <title>
```

#### Step G — Return

Return a single line in this format:
```
SUMMARY: <summary text> | SLUG: <slug>
```

If anything went wrong and you cannot produce a valid document, return:
```
ERROR: <brief description of what failed>
```

---

### 6. Update manifest

Parse the child agent's return value. If it begins with `ERROR:`, output the error message to the user and stop.

Otherwise extract the summary and slug. Read `~/research/manifest.json` and update it:
- If an entry for this slug already exists: update `date`, `tags`, `summary`, and `related` in-place
- If no entry exists: append a new object

Entry format:
```json
{"slug": "<slug>", "title": "<title>", "date": "<YYYY-MM-DD>", "tags": [...], "summary": "<summary>", "related": [<slug1>, <slug2>]}
```

Keep the array sorted by `date` descending. Write the updated array back to `~/research/manifest.json`.

### 7. Return

Output the summary text and slug so that calling skills can inject context:

```
Research complete. Summary: <summary>
Full doc: ~/research/<slug>/README.md
```
