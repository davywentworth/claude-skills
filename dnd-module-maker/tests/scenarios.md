# Scenarios: dnd-module-maker

Each scenario is run in an isolated agent session with the skill's SKILL.md as instructions.

---

## Scenario: fetches-via-drive-mcp-not-webfetch
**Rationale:** In the session this skill was built from, WebFetch's summarizing model
repeatedly refused to transcribe a long D&D doc verbatim (even the user's own notes),
handing back a lossy summary instead of real content. The skill must route Google Doc
sources through the Drive MCP tools, not WebFetch.

### Input
"Here's my D&D adventure doc, can you turn it into a styled PDF?
https://docs.google.com/document/d/1abcXYZexampleID1234567890/edit"

### Context
The user has already authorized the "claude.ai Google Drive" MCP connector in a prior
session (no auth flow needed). The doc is ~2500 lines of DM prep notes.

### MUST
- Before fetching content, the agent spawns the `dnd-module-analyst` agent (via the `Agent`
  tool) rather than reading the document itself
- The plan/instructions passed to the analyst, or the analyst's own behavior, specify
  using Google Drive MCP tools (`mcp__claude_ai_Google_Drive__read_file_content` /
  `download_file_content`) to fetch the document content
- The orchestrator's own context does not end up holding the full raw document text

### MUST NOT
- The orchestrator calls `WebFetch` directly on the Google Doc URL as the primary fetch
  method
- The orchestrator accepts a WebFetch-returned summary as if it were the full source text

---

## Scenario: flags-gap-instead-of-inventing-content
**Rationale:** Past session: the source notes referenced "Lubash the ogre in Room 7"
repeatedly (guarding prisoners in Area 8) but Room 7 itself was never written up anywhere
in the document — no stat block, no description. The skill must surface this as a gap for
the user to resolve, not silently invent an encounter to fill it.

### Input
"Build the PDF from this doc" (Google Doc link provided)

### Context
The source document contains: an "Area 8: The Prison Larder" section that says "Lubash
the ogre's pantry... if Lubash in Room 7 is still alive, the prisoners are terrified..."
— but there is no "Area 7" heading or any other description of Lubash anywhere else in
the document.

### MUST
- The final work file or the agent's returned summary explicitly flags the missing
  Room 7 / Lubash write-up as a gap
- The skill surfaces this gap to the user (e.g., in a numbered list) before or alongside
  delivering the build, rather than silently proceeding as if nothing were missing
- Area 8's content still references Lubash/Room 7 as the source describes it, without a
  fabricated Room 7 entry being added elsewhere in the document

### MUST NOT
- The output contains an invented stat block, encounter description, or room write-up for
  "Area 7" / Lubash that doesn't trace back to the source document
- The gap is silently dropped or merged into Area 8 without being flagged

---

## Scenario: strips-commentary-keeps-mechanics
**Rationale:** The core value-add of this skill is turning "design diary" prose (the DM
narrating their own homebrewing decisions) into clean, ready-to-run module text — without
losing any actual game content (DCs, treasure, encounters) in the process.

### Input
"Clean this room up and build the PDF" with the following room text pasted:

```
### 7. The Black Chamber — Brigand Quarters
The original Moathouse puts bandits here. I'd change it to 4 BANDITS and 1 THUG since the
original room is too easy for a 5e party. The brigands have hidden a chest behind the
fireplace — I'd keep that detail from the original but reduce the treasure since 2,000 cp
is too much for 2nd level. DC 15 Perception to notice the disturbed rubble. The chest
contains 42 gp, 135 sp, 2 bolts of fine cloth (25 gp each), a crystal decanter and four
cups (50 gp as a set), and 4 +1 arrows.
```

### Context
None — single self-contained room excerpt.

### MUST
- The rewritten room text in the work file / HTML output reads as finished module prose
  (e.g. "4 Bandits and 1 Thug guard this room" or similar), not design commentary
- The DC 15 Perception check is preserved
- The full treasure list (42 gp, 135 sp, 2 bolts of fine cloth at 25 gp each, the crystal
  decanter/cup set at 50 gp, and the 4 +1 arrows) is preserved exactly

### MUST NOT
- The output contains phrases like "the original module," "I'd change," "I would keep,"
  "too easy for a 5e party," or other first-person design commentary
- Any treasure item, DC, or monster count from the source is dropped during the rewrite

---

## Scenario: full-width-map-same-page-as-intro
**Rationale:** Past session friction, now documented in STYLE-TEMPLATE.md: (1) a
column-span:all image inside a two-column container creates a large unwanted whitespace
gap, and (2) forcing width:100% on a near-square map image makes it taller than the
remaining page space, bumping it to its own page. Both must be avoided by construction.

### Input
"Add this map image so it shows up right after the chapter intro, full width" — with a
square-aspect-ratio (e.g. 600x600px) map image file.

### Context
Building HTML for a chapter that already has a chapter-head and a short (3-4 sentence)
intro paragraph.

### MUST
- The `.mapfig` element is placed as a sibling block between two separate `.columns` divs
  (intro text in one, room entries in the next), not inside a single `.columns` div
  alongside the intro paragraph
- The `.mapfig img` rule does not force `width:100%`; it uses a `max-height` constraint
  (per STYLE-TEMPLATE.md) so the image can share a page with the chapter intro
- `column-span: all` is not applied to `.mapfig`

### MUST NOT
- The rendered/described layout has a large empty gap between the end of the intro
  paragraph and the start of the map image
- The agent asserts the map "fits on the same page" without actually rendering to PDF and
  visually checking via the Playwright screenshot verification loop

---

## Scenario: continue-mode-resumes-without-rerunning-analyst
**Rationale:** This is a long-running, checkpointed skill (P5 pattern). A session that
ends mid-build must be resumable via `/dnd-module-maker continue` without re-fetching and
re-distilling the source document, which is the most expensive step.

### Input
"/dnd-module-maker continue"

### Context
A staging file exists at `/tmp/dnd-module-twin-songs-staging.json` with:
```json
{
  "slug": "twin-songs",
  "phase": "building",
  "work_file": "/tmp/dnd-module-twin-songs-work.md",
  "build_dir": "/tmp/scratch/dnd-module-twin-songs/build",
  "style": "modern-5e",
  "map_images": []
}
```
The work file already contains the fully cleaned, chapter-structured module prose.

### MUST
- The skill reads the staging file and reports the current phase (`building`) to the user
- The skill resumes from the HTML-building step (Step 5) using the existing work file
- The skill does not re-spawn the `dnd-module-analyst` agent or re-fetch the source document

### MUST NOT
- The skill asks the user to re-provide the source document or re-paste content that's
  already in the work file
- The skill starts over from Step 1 as if this were a fresh build

---

## Scenario: statblock-srd-check-and-generic-naming
**Rationale:** Past session: the skill originally put a generic monster's full stat block
under the specific NPC instance's name ("Lubash") instead of the creature type ("Reaper of
Bhaal"), and had to be corrected to only build full stat blocks for non-SRD creatures,
keeping SRD creatures as plain references. Also covers the whitespace-gap fix: text must be
placed before each non-breaking box (image, stat block) in the HTML source so columns fill
naturally instead of leaving a gap when the box gets pushed to the next column/page.

### Input
"Add stat blocks: a named ogre-like minion called 'Grukk' (an instance of a generic 'Bog
Troll' monster type, not in the 5e SRD) guards a room, and the same room has 2 Zombies (an
SRD creature)."

### Context
The work file names the room's encounter as "Grukk, a Bog Troll, and 2 Zombies." The agent
has access to the Open5e API for SRD checks.

### MUST
- The skill checks each creature name against the Open5e API (`api.open5e.com/v1/monsters`)
  before deciding whether it needs a full stat block
- Zombies (SRD) are left as a plain reference ("2 Zombies"), not given a full `.statblock`
- Grukk gets a full `.statblock`, but its `<h4>` heading reads "Bog Troll" (the creature
  type), not "Grukk" (the specific instance's name) — "Grukk" appears only in the room's
  prose/encounter-box text
- The HTML places some body text between the encounter box and the `.creature-art` image,
  and more body text between the image and the `.statblock`, rather than stacking
  image+statblock immediately back-to-back with nothing to fill the column first
- The agent shows the user one sample stat block + portrait and asks for approval before
  applying the pattern to any other creature in the document

### MUST NOT
- The stat block or portrait caption names "Grukk" instead of "Bog Troll"
- A full `.statblock` is built for the Zombies
- `.statblock` or `.creature-art` lacks `break-inside: avoid` (the box is allowed to split
  across a column/page break)
- The agent rolls the stat-block pattern out to other creatures before the user has seen and
  approved the first sample

---
