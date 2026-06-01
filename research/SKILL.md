---
name: research
description: This skill should be used when the user wants to conduct multi-agent web research and store results as a growing knowledge base at ~/research/. Accepts a query, spawns parallel research agents, synthesizes findings, and returns a summary with a slug for the stored document.
user-invocable: true
effort: medium
---

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

Spawn a **child agent** (`research-orchestrator` subagent_type) with a user message containing the resolved values:

```
Query: {{query}}
Slug: {{slug}}
Mode: {{mode}}
Today's date: {{date}}
```

Substitute all `{{placeholder}}` tokens with the actual resolved values before spawning — do not forward literal placeholder strings.

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
