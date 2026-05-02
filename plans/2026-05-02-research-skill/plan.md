# Research Skill — Implementation Plan

**Goal:** Build a `/research` skill with a child→grandchildren multi-agent web research pattern, a growing knowledge base at `~/research/`, a static local viewer, and auto-integration into `/brainstorm`.
**Design doc:** `plans/2026-05-02-research-skill/brainstorm.md` — read this first for architecture and decisions.

**Assumptions and boundaries:**
- In scope: `/research` skill, viewer (`index.html` + `manifest.json`), `research-view` alias, `/brainstorm` integration, memory entry
- Not in scope (later): self-directed integration in `/issue plan`, `/devils-advocate`, `/interview`
- Relies on: `WebSearch` and `WebFetch` tools being available to grandchild agents; marked.js CDN availability for viewer

---

## Stages

### Stage 1: Knowledge base scaffolding
Create `~/research/` directory and initial `manifest.json`.

**Files:**
- `~/research/manifest.json` — empty array `[]`

**Done:** Directory exists, manifest is valid JSON.

---

### Stage 2: `/research` skill SKILL.md
Write `/Users/davy/dev/claude-skills/research/SKILL.md` covering:
- Invocation (`/research <query>`, `--no-research` flag)
- Slug derivation logic (lowercase, hyphens, max 40 chars)
- Slug similarity check against manifest before writing
- Child agent instructions: read existing doc if present, decompose into 0–6 subtopics, spawn grandchildren in parallel
- Grandchild agent instructions: 3–5 WebSearch queries, WebFetch top results, return structured findings
- Synthesis: child merges findings, incremental enrichment rules (never overwrite, append references, update date)
- Output format: frontmatter schema + section structure + `[N]` citation style
- Post-write: update `manifest.json` with new/updated entry `{slug, title, date, tags, summary}`

**Depends on:** Stage 1

**Done:** Skill file written and symlinked to `~/.claude/commands/research`.

---

### Stage 3: Static viewer
Write `~/research/index.html` — a self-contained single-page viewer:
- On load: `fetch('manifest.json')` → build sidebar list (sorted by date, filterable by keyword/tag)
- On topic click: `fetch('<slug>/README.md')` → strip `---` frontmatter block → pass to `marked.parse()` → render in main panel
- marked.js loaded from CDN (`https://cdn.jsdelivr.net/npm/marked/marked.min.js`)
- All links in rendered content: `target="_blank"`
- Styling: minimal, readable, no external CSS frameworks

**Depends on:** Stage 1

**Done:** `index.html` opens correctly when served via `python -m http.server`; sidebar populates from manifest; a test MD file renders correctly with frontmatter stripped.

---

### Stage 4: `research-view` alias
Append to `~/.bash_profile`:
```bash
alias research-view='cd ~/research && python -m http.server 8765 & sleep 1 && open http://localhost:8765'
```

**Done:** `source ~/.bash_profile` and `research-view` opens browser at `http://localhost:8765`.

---

### Stage 5: `/brainstorm` integration
Modify `/Users/davy/dev/claude-skills/brainstorm/SKILL.md`:
- Add Step 0 at the top of Phase 1, before "Read project context"
- Step 0: check for `--no-research` flag; if absent, call `/research <topic>` via Skill tool; inject the returned `summary` into context with a note that full doc is at `~/research/<slug>/README.md`

**Depends on:** Stage 2

**Done:** `/brainstorm some topic` triggers research; `/brainstorm --no-research some topic` skips it.

---

### Stage 6: Memory entry
Write a reference memory file at `~/.claude/projects/-Users-davy/memory/research_knowledge_base.md` and add a pointer to `MEMORY.md`:
- Describes `~/research/` structure
- Notes: read `manifest.json` first, scan `summary` + `tags` for relevance, then fetch matching `README.md` files
- Instructs Claude to check this knowledge base at the start of planning sessions

**Done:** Memory file written and indexed in MEMORY.md.

---

### Stage 7: Commit and symlink
```bash
cd /Users/davy/dev/claude-skills
git add research/SKILL.md
git commit -m "Add research skill"
git push
ln -sf /Users/davy/dev/claude-skills/research ~/.claude/commands/research
```

**Depends on:** Stage 2

**Done:** Skill committed, pushed, symlinked.
