---
name: dnd-module-maker
description: This skill should be used when the user wants to turn a messy D&D/TTRPG adventure document — DM prep notes, often mixing references to a published module with homebrew rewrite commentary like "I'd keep X from the original, change Y" — into a polished, print-quality PDF styled like an official sourcebook. Also use for `/dnd-module-maker continue` to resume an in-progress build.
user-invocable: true
---

Turn messy DM notes into a module a DM can actually run from the table: a styled,
print-quality PDF with two-column layout, boxed read-aloud text, encounter/treasure
callout boxes, and clean final prose instead of design-diary commentary.

This is a long-running, multi-phase skill (fetch/distill → user review → build → render →
visually verify → iterate → deliver). It checkpoints to a staging file so a session that
ends mid-build can resume with `/dnd-module-maker continue` instead of starting over.

## Step 0: Continue mode

If invoked as `/dnd-module-maker continue`, look for a staging file at
`/tmp/dnd-module-*-staging.json`. If one exists, read it, report the current `phase` to
the user, and resume from there (skip straight to the matching step below — do not
re-run the analyst agent if `phase` is already `distilled` or later). If none exists, tell
the user there's nothing to resume and ask if they want to start a new build.

## Step 1: Gather inputs

Figure out, asking only for what's missing:

1. **Source** — a Google Doc link, a local file path, or pasted text. If the user mentions
   skipping pages or starting at a specific heading, or describes structural quirks (e.g.
   "there are two levels," "the map is rotated"), capture that as `USER_NOTES` verbatim —
   the analyst agent needs it.
2. **Style** — ask via `AskUserQuestion` if not specified: a modern-5E-style look (two-column,
   red/gold accents, boxed callouts — the default in `STYLE-TEMPLATE.md`), a classic
   old-school single-column look, or a minimalist modern look. Only the color palette and a
   few typographic choices change; the structural techniques in `STYLE-TEMPLATE.md` (full
   bleed, column-span avoidance, image sizing) apply regardless of style.
3. **Map images**, if any — ask whether the user has separate map image files to include
   (exported PNGs, etc.) and where to find them. Don't try to extract or auto-rotate images
   embedded in the source doc itself unless asked; place user-supplied map files as given.

Derive a `slug` from the title (lowercase-hyphenated) for file naming.

## Step 2: Front-load the fetch + editorial pass

Write the staging file immediately, before spawning the agent (progressive checkpointing —
don't wait until the agent returns):

```json
{
  "slug": "<slug>",
  "phase": "distilling",
  "work_file": "/tmp/dnd-module-<slug>-work.md",
  "build_dir": "<scratchpad>/dnd-module-<slug>/build",
  "style": "<chosen style>",
  "map_images": ["<path>", "..."]
}
```

Spawn the `dnd-module-analyst` agent (`Agent` tool, not a fresh general-purpose agent) with
`SOURCE`, `WORK_FILE`, and `USER_NOTES`. This agent does the heavy read (Drive MCP fetch,
never WebFetch — see the agent's own notes on why) and the full editorial rewrite pass,
writing clean module prose to the work file and returning only a distilled structural
summary. **Do not read the source document yourself** — that defeats the point of
front-loading it.

Update the staging file's `phase` to `"distilled"` when the agent returns.

## Step 3: Surface gaps, confirm structure

From the agent's returned summary, present any flagged gaps/ambiguities to the user as a
numbered list — don't silently paper over them or invent content to fill them. Confirm the
title/subtitle suggestion and chapter structure. Resolve anything the user wants changed by
editing the work file directly (it's plain markdown) rather than re-running the agent,
unless the fix requires re-reading the source.

## Step 4: Review the cleaned prose

Invoke `plannotator-annotate` on the `WORK_FILE` so the user can review the editorial pass
before it's locked into HTML — catching a bad commentary-strip here is much cheaper than
after the PDF is built. Remind the user they need to interact with at least one element
(e.g. 👍 on a section) before closing, or the process hangs. Apply any requested edits
directly to the work file.

## Step 5: Build the HTML/CSS

Read `STYLE-TEMPLATE.md` (in this skill's directory) in full — it has the CSS, HTML
structure patterns, and a list of known Chrome print-engine failure modes with their
fixes. Adapt the color palette to the chosen style (Step 1) but keep the structural
techniques (full-bleed wrapper, sibling-block image placement, image max-height) as-is;
they're fixes for real rendering bugs, not stylistic choices.

Create `<build_dir>/style.css` and `<build_dir>/index.html`. Read the work file in chunks
(`Read` with `offset`/`limit` for a long document) rather than pulling it all into context
at once if it's large. Copy any user-supplied map images into `<build_dir>/`.

Update the staging file's `phase` to `"building"`.

## Step 6: Render and verify

Render with the headless Chrome command in `STYLE-TEMPLATE.md`. Then run the verification
loop described there: `npx -y serve` the build dir in the background, `browser_navigate`
to the **http:// URL of the PDF** via Playwright MCP (not `file://`, which Playwright
blocks), screenshot, and `Read` the screenshot to actually check it — don't claim something
renders correctly without looking.

Check specifically for the known failure modes listed in `STYLE-TEMPLATE.md` (white border
around the background, whitespace gaps before images, images bumped to their own page,
title-page overflow, split dialogue quotes) before reporting the build as done.

Update the staging file's `phase` to `"rendered"`.

## Step 7: Iterate on feedback

When the user asks for visual changes, make the CSS/HTML edit, re-render, and re-verify
with the same screenshot loop — don't just assert the fix worked. If a fix is a structural
one (e.g. the column-span whitespace gap, or an image getting pushed to its own page),
check whether `STYLE-TEMPLATE.md` already documents it before re-deriving a solution from
scratch, and consider adding a newly-discovered gotcha back into that file for next time.

## Step 8: Deliver

Copy the final PDF to `~/Downloads/`, not just the build directory (which is
session-scoped). Deliver with `SendUserFile` (`display: "render"`) and `open` it locally.
Update the staging file's `phase` to `"delivered"`, then delete the staging file — the
build is complete and there's nothing left to resume.
