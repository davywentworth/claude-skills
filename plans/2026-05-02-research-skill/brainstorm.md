# Research Skill — Brainstorm

## Goal

A `/research` skill that conducts multi-agent web research and stores results in a growing knowledge base at `~/research/`. Integrated into planning skills by default. Results are human-consumable via a static viewer served with a one-liner alias, and machine-parsable by Claude via frontmatter and a manifest index.

---

## Architecture Overview

Four components:
1. **`/research` skill** — orchestrates a child → grandchildren agent pattern
2. **Static viewer** — `~/research/index.html` + `manifest.json`, served via `research-view` alias
3. **Planning integration** — `/brainstorm` auto-calls `/research`; other skills let the research agent self-direct
4. **Claude parsability** — frontmatter + manifest enable fast relevance scanning without reading full docs

---

## Component 1: `/research` Skill

### Invocation
- `/research <query>` — manual
- Called programmatically by planning skills (passes topic as query)
- `--no-research` flag skips execution when called from planning skills

### Slug Derivation
- Lowercase, hyphen-separated, max 40 chars, derived from query
- Before writing: scan `manifest.json` for similar titles/tags
- If a close match exists: offer to expand existing doc or create new slug

### Agent Pattern

```
Main skill
  └── Child agent (Orchestrator)
        ├── Reads existing doc if slug exists (for incremental enrichment)
        ├── Decomposes query into 0–6 subtopics based on complexity
        └── Grandchild agents (one per subtopic, spawned in parallel)
              ├── 3–5 WebSearch queries per subtopic
              ├── WebFetch top results for full content
              └── Returns structured findings block to child
```

- **0 grandchildren**: topic is simple enough for the orchestrator to handle alone
- **1–6 grandchildren**: each owns one subtopic end-to-end
- Child synthesizes all grandchild findings into the final doc

### Incremental Enrichment
- If `~/research/<slug>/README.md` exists: child reads it first, then expands — never overwrites existing content
- New references appended without duplicating
- `date` frontmatter updated to latest run
- New subtopic sections added below existing ones; existing sections expanded in-place

### Output Format

```markdown
---
title: Claude Plugin Development
date: 2026-05-02
tags: [claude, plugins, sdk]
query: "how to write claude plugins"
summary: "Claude plugins use a SKILL.md convention with a directory structure symlinked into ~/.claude/commands/. Skills are invoked via the Skill tool. Key patterns: imperative tone, step-by-step, reuse existing skills rather than reproducing inline."
related: [claude-api, agent-sdk]
---

## Overview
...

## [Subtopic sections]
...

## Key Takeaways
- ...

## References
[1]: https://...
[2]: https://...
```

All inline citations use `[N]` notation linking to the numbered references section. Frontmatter is stripped by the viewer before rendering so users never see raw YAML.

---

## Component 2: Static Viewer

### File Structure
```
~/research/
  index.html        ← viewer: sidebar + main panel, marked.js CDN, strips frontmatter before render
  manifest.json     ← [{slug, title, date, tags, summary}] — updated each research run
  <slug>/
    README.md
```

### index.html
- Sidebar: all research topics sorted by date, filterable by tag/keyword (plain JS string match)
- Main panel: renders selected `README.md` via marked.js (CDN), stripping `---` frontmatter block first
- All reference links open in new tab
- Reads `manifest.json` on load to build sidebar, then fetches individual `README.md` on click

### research-view alias
Written to `~/.bash_profile` during setup:
```bash
alias research-view='cd ~/research && python -m http.server 8765 & sleep 1 && open http://localhost:8765'
```

Requires local server because `fetch()` is blocked on `file://` URLs. The alias handles this transparently.

---

## Component 3: Planning Integration

### `/brainstorm` (auto-trigger)
- New Step 0 added to brainstorm SKILL.md before all existing steps
- Checks for `--no-research` flag — skips if present
- Calls `/research <topic>` with the brainstorm topic as query
- Injects the research `summary` into brainstorm context before design begins

### `/issue plan`, `/devils-advocate`, `/interview` (self-directed)
- No automatic trigger
- These skills note that the agent may conduct supplemental web research if the topic warrants it
- Handled in a later iteration

---

## Component 4: Claude Parsability

### How Claude finds relevant research
1. Reads `~/research/manifest.json` — scans `summary` and `tags` to identify relevant docs
2. Fetches only the `README.md` files that match
3. Uses `related` frontmatter field to pull connected docs

### Memory entry
A reference memory entry added to `~/research/` describing the structure, so future sessions know to check manifest first before planning.

---

## Decisions Made

| Decision | Choice | Reason |
|---|---|---|
| Frontmatter location | In `README.md`, stripped by viewer | Single self-contained file; universal convention; metadata travels with the doc |
| Viewer hosting | Local server via alias | No backend; avoids `file://` CORS; single command |
| Viewer rendering | marked.js via CDN | Zero bundling; embedded in index.html |
| Slug collision | Expand existing | Knowledge accumulates over time |
| Similar slug detection | Scan manifest tags/titles | Prevents fragmented duplicate docs |
| Planning integration | Auto in brainstorm; self-directed elsewhere | Brainstorm is the planning entry point |
| Claude access | manifest.json + frontmatter summary | Fast relevance scan without reading full docs |
