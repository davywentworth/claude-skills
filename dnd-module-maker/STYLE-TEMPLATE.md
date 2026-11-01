# Style Template — D&D Module Maker

Read this file before building the HTML/CSS in Step 3. Use the CSS and structure below as
the starting point — adapt the color palette to the chosen style variant (see SKILL.md),
but keep the component classes and the full-bleed/pagination techniques as-is. They encode
hard-won fixes for real Chrome print-engine bugs; don't re-derive them from scratch.

---

## CSS (save as `style.css`, link from `index.html`)

```css
@import url('https://fonts.googleapis.com/css2?family=Cinzel:wght@500;700;900&family=Cinzel+Decorative:wght@700&family=EB+Garamond:ital,wght@0,400;0,500;0,600;0,700;1,400;1,500&display=swap');

:root{
  --maroon:#5c1a1a;
  --maroon-dark:#3e1010;
  --gold:#9c7a2e;
  --gold-light:#c8a850;
  --cream:#fbf6ea;
  --cream-box:#f4ead2;
  --ink:#241c14;
  --rule:#b5a26b;
}

/* @page margin MUST be 0 — see "Full-bleed background" below for why. */
@page{ size: letter; margin: 0; }

*{ box-sizing:border-box; }

html,body{
  margin:0; padding:0;
  background:#fff;
  color:var(--ink);
  font-family:'EB Garamond', 'Palatino', Georgia, serif;
  font-size:12.4pt;
  line-height:1.33;
}

/* ---------- Full-bleed page wrapper ----------
   @page margin is 0, so this wrapper supplies the visual margin itself.
   box-decoration-break:clone makes Chrome's print engine re-apply its
   background AND padding on every printed page (not just once for the
   whole flowed box), so the background bleeds to the true edge on every
   page while text still sits inset. Wrap your ENTIRE <body> content in
   a single div with this class — do not rely on @page margin + a plain
   body/html background, it will NOT bleed (tested: neither html{background}
   nor a position:fixed full-viewport layer paints into the @page margin
   area — only this wrapper technique works). */
.pagewrap{
  background:var(--cream);
  background-image:
    radial-gradient(ellipse at top left, rgba(156,122,46,0.06), transparent 60%),
    radial-gradient(ellipse at bottom right, rgba(92,26,26,0.05), transparent 60%);
  padding: 0.55in 0.5in 0.6in 0.5in;
  box-decoration-break: clone;
  -webkit-box-decoration-break: clone;
}

/* ---------- Title page ---------- */
.titlepage{
  page-break-after: always;
  height: 9.55in;              /* tuned to fit ONE page after .pagewrap padding — see note */
  display:flex;
  flex-direction:column;
  align-items:center;
  justify-content:center;
  text-align:center;
  border: 3px double var(--maroon);
  margin: 0.1in;
  padding: 0.6in 0.35in;
  background: linear-gradient(180deg, rgba(92,26,26,0.04), rgba(156,122,46,0.06));
}
.titlepage .kicker{ font-family:'Cinzel', serif; letter-spacing:0.3em; font-size:13.2pt; color:var(--maroon); margin-bottom:0.35in; }
.titlepage h1{ font-family:'Cinzel Decorative','Cinzel', serif; font-size:46pt; color:var(--maroon-dark); margin:0 0 0.15in 0; text-shadow:1px 1px 0 rgba(156,122,46,0.35); line-height:1.08; }
.titlepage .sub{ font-family:'Cinzel', serif; font-size:19.2pt; color:var(--gold); letter-spacing:0.08em; margin-bottom:0.4in; }
.titlepage .divider{ width:60%; border-top:2px solid var(--gold-light); border-bottom:1px solid var(--gold-light); height:4px; margin:0.3in 0; }
.titlepage .blurb{ font-style:italic; font-size:15pt; max-width:5.9in; color:#43301a; }
.titlepage .credit{ margin-top:0.6in; font-family:'Cinzel', serif; font-size:11.4pt; letter-spacing:0.15em; color:var(--maroon); text-transform:uppercase; }

/* ---------- Chapter openers ---------- */
.chapter{ page-break-before: always; }
.chapter-head{ column-span:all; text-align:center; margin-bottom:0.25in; padding-bottom:0.12in; border-bottom:3px double var(--maroon); }
.chapter-head .eyebrow{ font-family:'Cinzel', serif; font-size:12pt; letter-spacing:0.25em; color:var(--gold); text-transform:uppercase; }
.chapter-head h1{ font-family:'Cinzel Decorative','Cinzel',serif; font-size:36pt; color:var(--maroon-dark); margin:0.05in 0; }
.chapter-head .deck{ font-style:italic; font-size:13.8pt; color:#4a3820; }

.columns{ column-count:2; column-gap:0.38in; }

/* ---------- Section / area (room) headers ---------- */
.area{ margin-bottom:0.14in; }
/* break-inside:avoid is scoped to JUST .area-head (the badge + title row),
   not the whole .area block. avoid-page alone isn't enough — Chrome treats
   column breaks as a separate fragmentation context, so the circular
   .area-num badge can still get sliced mid-render at a column boundary and
   partially re-render on the next column. Putting break-inside:avoid on
   the whole .area instead of just .area-head "fixes" that but is WRONG:
   it forces the entire room entry (flavor box, encounter, paragraphs,
   treasure) to jump to the next column/page as one unit whenever it
   doesn't fit the remaining space, leaving a large whitespace gap at the
   bottom of the column it left. Keep the protection scoped to the small
   header row only; break-after:avoid keeps it glued to the first line of
   body text that follows. */
.area-head{ display:flex; align-items:center; gap:0.12in; margin:0.1in 0 0.08in 0; break-inside:avoid; break-after:avoid; }
.area-num{ flex:0 0 auto; background:var(--maroon); color:var(--cream); font-family:'Cinzel', serif; font-weight:700; font-size:15.6pt; width:0.5in; height:0.5in; border-radius:50%; display:flex; align-items:center; justify-content:center; border:2px solid var(--gold-light); }
.area-title{ font-family:'Cinzel', serif; font-weight:700; font-size:15.6pt; color:var(--maroon-dark); text-transform:uppercase; letter-spacing:0.02em; border-bottom:1.5px solid var(--rule); flex:1; padding-bottom:2px; }
.area-sub{ font-family:'Cinzel', serif; font-weight:500; font-size:13.2pt; color:var(--maroon); margin:0.12in 0 0.05in 0; letter-spacing:0.02em; border-bottom:1px dotted var(--rule); }

/* ---------- Boxed read-aloud flavor text ---------- */
.flavor{ background:var(--cream-box); border-left:4px solid var(--maroon); border-top:1px solid var(--rule); border-bottom:1px solid var(--rule); border-right:1px solid var(--rule); padding:0.09in 0.14in; font-style:italic; font-size:12pt; margin:0.06in 0 0.1in 0; break-inside:avoid; }

/* ---------- Encounter / treasure boxes ---------- */
.encounter{ background:#eee2ca; border:1.5px solid var(--maroon); padding:0.07in 0.12in; margin:0.08in 0; break-inside:avoid; font-size:11.5pt; }
.encounter .label{ font-family:'Cinzel', serif; font-weight:700; font-size:10pt; letter-spacing:0.14em; color:var(--maroon); display:block; margin-bottom:2px; }
.treasure{ background:#f5ecce; border:1.5px solid var(--gold); padding:0.07in 0.12in; margin:0.08in 0; break-inside:avoid; font-size:11.5pt; }
.treasure .label{ font-family:'Cinzel', serif; font-weight:700; font-size:10pt; letter-spacing:0.14em; color:#7a5c17; display:block; margin-bottom:2px; }
.treasure ul{ margin:0.03in 0 0 0.16in; padding:0; }
.treasure li{ margin-bottom:1px; }

/* ---------- Inline semantic tags ---------- */
.chk{ font-weight:700; color:var(--maroon-dark); font-variant:small-caps; letter-spacing:0.01em; }   /* DC/skill checks */
.roll{ font-weight:700; color:#5c1a1a; background:rgba(156,122,46,0.14); padding:0 3px; border-radius:2px; }  /* dice damage e.g. 1d6 */
.npc{ font-weight:700; }
span.gp{ font-weight:700; color:#7a5c17; }

blockquote.say{ margin:0.06in 0.02in 0.08in 0.06in; padding-left:0.14in; border-left:2px solid var(--gold); font-style:italic; color:#3a2c18; }
/* IMPORTANT: without this, a quote paragraph can split awkwardly across a column/page break. */
blockquote.say p{ break-inside: avoid; }

h3.mini{ font-family:'Cinzel', serif; font-weight:700; font-size:11.5pt; text-transform:uppercase; letter-spacing:0.08em; color:var(--maroon); margin:0.09in 0 0.03in 0; break-after:avoid; }

p{ margin:0 0 0.07in 0; text-align:justify; hyphens:auto; }
ul,ol{ margin:0.02in 0 0.08in 0.18in; padding:0; }
li{ margin-bottom:1px; }

.dropcap:first-letter{ font-family:'Cinzel Decorative','Cinzel',serif; font-size:2.6em; float:left; line-height:0.78; padding-right:4px; color:var(--maroon-dark); }

table.dmtable{ width:100%; border-collapse:collapse; font-size:10.3pt; margin:0.08in 0 0.12in 0; }
table.dmtable th{ background:var(--maroon); color:var(--cream); font-family:'Cinzel',serif; font-size:9.1pt; letter-spacing:0.06em; text-transform:uppercase; padding:3px 5px; text-align:left; }
table.dmtable td{ border-bottom:1px solid var(--rule); padding:3px 5px; vertical-align:top; }

/* ---------- Full-width map/figure images ----------
   Do NOT put this inside a .columns container with column-span:all — a
   spanning element forces the columns above it to balance/equalize height
   FIRST, which creates large unwanted whitespace gaps before the image.
   Instead structure each chapter as separate SIBLING blocks:
     <div class="columns">intro paragraph(s)</div>
     <div class="mapfig">image</div>
     <div class="columns">room entries...</div>
   This keeps the image a plain block between two column regions instead
   of a spanning element inside one. */
.mapfig{ margin:0.22in 0 0.2in 0; break-inside:avoid; text-align:center; }
.mapfig img{
  display:inline-block;
  width:auto; height:auto;
  max-width:100%;
  /* Cap height, don't force width:100%. A near-square source image at
     true full content-width (~7.5in) can be TALLER than the remaining
     page space after the chapter header + intro text, which pushes it
     onto its own page. Capping height keeps it reliably on the same
     page; width then scales down to fit, which still reads as "full
     width" for any image wider than it is tall. */
  max-height:5.4in;
  background:#fff;
  border:2px solid var(--maroon);
  padding:6px;
}
.mapfig .caption{ font-family:'Cinzel', serif; font-size:10pt; letter-spacing:0.06em; color:var(--maroon); text-align:center; margin-top:0.06in; font-style:normal; }

/* ---------- Monster stat blocks (non-SRD creatures only — see "Stat
   blocks: when to include" below) ---------- */
.statblock{
  /* Never splits -- break-inside:avoid pushes the whole box to the next
     column/page as a unit if it doesn't fit the remaining space, rather
     than cutting it mid-box. This alone can create a large whitespace gap
     if nothing else fills the column first -- see "Stat blocks must never
     wrap" below for the fix (put real body text before it in the HTML). */
  background: var(--cream-box);
  border: 2px solid var(--maroon);
  padding: 0.12in 0.16in;
  margin: 0.1in 0 0.14in 0;
  break-inside: avoid;
  font-size: 10.6pt;
}
/* Escape valve: if a stat block is too tall to ever fit a single column
   (lots of traits/actions), use this as a sibling block OUTSIDE .columns,
   same pattern as .mapfig, so it spans both columns at the page bottom. */
.statblock.wide{ columns: 2; column-gap: 0.38in; }
.statblock h4{ font-family:'Cinzel', serif; font-weight:700; font-size:13.6pt; color:var(--maroon-dark); text-transform:uppercase; letter-spacing:0.02em; margin:0; }
.statblock .sb-subtitle{ font-style:italic; font-size:10pt; color:#4a3820; margin:0 0 0.05in 0; }
.statblock hr{ border:none; border-top:2px solid var(--maroon); margin:0.06in 0; }
.statblock .sb-topline{ margin:0.02in 0; }
.statblock .sb-scores{ width:100%; border-collapse:collapse; margin:0.06in 0; text-align:center; font-size:9.6pt; }
.statblock .sb-scores th{ font-family:'Cinzel', serif; font-size:8.6pt; letter-spacing:0.08em; color:var(--maroon); padding:1px 2px; border-bottom:1px solid var(--rule); }
.statblock .sb-scores td{ padding:1px 2px; }
.statblock .sb-meta p{ margin:0.02in 0; }
.statblock .sb-traits p, .statblock .sb-actions p{ margin:0.04in 0; text-align:left; }
.statblock .sb-actions-head{ font-family:'Cinzel', serif; font-weight:700; font-size:10.6pt; color:var(--maroon-dark); text-transform:uppercase; letter-spacing:0.06em; border-bottom:1px solid var(--rule); margin:0.08in 0 0.03in 0; }
.statblock .sb-source{ font-size:8.8pt; font-style:italic; color:#6a5a3a; margin-top:0.06in; }

/* ---------- Standalone creature portrait ----------
   Deliberately separate from .statblock and labeled with the creature TYPE
   only (never a specific NPC's name, unless that NPC truly is one-of-a-kind
   -- see "Generic type vs. unique individual" below) so the same art +
   label can be reused anywhere else that creature type appears. */
.creature-art{
  width: 100%; /* full width of its column (or the 2-col span if .wide) */
  margin: 0.05in 0 0.1in 0;
  text-align:center;
  break-inside: avoid; /* protects image+caption as one unit; see gotcha below */
}
.creature-art.wide img{ max-height: 4.2in; } /* for landscape art spanning 2 columns, same as .mapfig */
.creature-art.wide .frame{ display:inline-block; }
.creature-art .frame{ position: relative; border: 3px double var(--maroon); padding: 7px; background: #fff; display:block; }
.creature-art .frame::before{ content:""; position:absolute; inset: 3px; border: 1px solid var(--gold-light); pointer-events:none; }
.creature-art img{ display:block; width:100%; height:auto; }
.creature-art .caption{ font-family:'Cinzel', serif; font-size:8.6pt; letter-spacing:0.08em; color:var(--maroon); text-transform:uppercase; margin-top:0.05in; }

/* ---------- Cross-reference jump links (e.g. a monster name in prose
   pointing at its stat block elsewhere in the document) ---------- */
a.xref{ color: var(--maroon); font-weight: 700; text-decoration: underline; text-decoration-color: var(--gold); text-decoration-thickness: 1px; }
```

---

## HTML structure

### Page wrapper (wrap the entire `<body>` content)

```html
<body>
<div class="pagewrap">
  <!-- title page, chapters -->
</div>
</body>
```

### Title page

```html
<div class="titlepage">
  <div class="kicker">[Eyebrow line, e.g. source/setting]</div>
  <h1>[Module Title]</h1>
  <div class="sub">[Subtitle, e.g. level range]</div>
  <div class="divider"></div>
  <div class="blurb">[One or two sentence hook]</div>
  <div class="credit">[Campaign Notes &middot; DM name(s)]</div>
</div>
```

### Chapter (note the THREE sibling blocks — see the full-width image comment above)

```html
<div class="chapter">
  <div class="chapter-head">
    <div class="eyebrow">Chapter [N]</div>
    <h1>[Chapter Title]</h1>
    <div class="deck">[One-line chapter deck]</div>
  </div>

  <div class="columns">
    <p class="dropcap">[Chapter intro paragraph(s).]</p>
  </div>

  <!-- Only include this block if there's a map for this chapter -->
  <div class="mapfig">
    <img src="[map-filename].png" alt="[Map description]">
    <div class="caption">[Map Caption]</div>
  </div>

  <div class="columns">
    <!-- AREA 1, AREA 2, ... -->
  </div>
</div>
```

### Room/area entry

```html
<div class="area" id="[chapter-prefix][N]">
  <div class="area-head">
    <div class="area-num">[N]</div>
    <div class="area-title">[Room Name]</div>
  </div>
  <div class="flavor">[Boxed read-aloud sensory description.]</div>
  <div class="encounter"><span class="label">Encounter</span>[Monster/NPC summary, if any]</div>
  <p>[Body text. Wrap checks like <span class="chk">DC 12 Wisdom (Perception)</span> and
  damage like <span class="roll">1d6 bludgeoning</span>.]</p>
  <h3 class="mini">[Subheading, e.g. Tactics / What They Know]</h3>
  <p>[More body text.]</p>
  <div class="treasure"><span class="label">Treasure</span>
    <ul><li>[Item, value]</li></ul>
  </div>
</div>
```

### Stat blocks: when to include

Only build a full `.statblock` for creatures NOT in the SRD (check via the Open5e API,
`api.open5e.com/v1/monsters/?search=<name>` — a `wotc-srd` document slug means it's SRD).
For SRD creatures (Zombie, Bandit, Ghoul, Giant Spider, etc.), just reference them by name
("6 Zombies") — any GM can look those up freely, so a full box is unnecessary bulk. Prefer
the 2024/5.5e SRD revision if one exists for a creature that needs a box; if the source is a
specific non-SRD adventure (e.g. *Descent into Avernus*), there usually isn't one, so use the
original.

### Stat block + portrait: placement pattern

```html
<div class="encounter"><span class="label">Encounter</span>
  [NPC name], a [<a class="xref" href="#statblock-[type-slug]">creature type</a>]
  (AC [x], HP [y], Initiative [+z]), [one-line context].
</div>

<p>[Body text that would otherwise follow the stat block — tactics, a trigger
condition, anything narratively tied to this room. Put ENOUGH of it here, before
the portrait, that the column fills up naturally.]</p>

<div class="creature-art">
  <div class="frame"><img src="[creature-type-slug].png" alt="[Creature Type]"></div>
  <div class="caption">[Creature Type]</div> <!-- generic type name, see rule below -->
</div>

<p>[More body text, if any — this is what sits between the portrait and the stat
block. Can be empty if the first paragraph above already filled the column.]</p>

<div class="statblock" id="statblock-[type-slug]">
  <h4>[Creature Type, or the NPC's own name if unique]</h4>
  <div class="sb-subtitle">[Size type (race), alignment]</div>
  <hr>
  <p class="sb-topline"><b>Armor Class</b> [x]</p>
  <p class="sb-topline"><b>Hit Points</b> [y] ([xdy+z])</p>
  <p class="sb-topline"><b>Speed</b> [x] ft.</p>
  <table class="sb-scores">
    <tr><th>STR</th><th>DEX</th><th>CON</th><th>INT</th><th>WIS</th><th>CHA</th></tr>
    <tr><td>[x (+y)]</td><!-- ... --></tr>
  </table>
  <hr>
  <div class="sb-meta"><p><b>Challenge</b> [x] ([y] XP)</p></div>
  <div class="sb-traits"><p><b><i>[Trait name].</i></b> [Trait text.]</p></div>
  <div class="sb-actions-head">Actions</div>
  <div class="sb-actions"><p><b><i>[Action name].</i></b> [Action text.]</p></div>
  <p class="sb-source">[Creature], <i>[Sourcebook]</i> (CR [x]). [Note what's
  verified from an authoritative source vs. reconstructed/inferred — never present
  a guess as confirmed text.]</p>
</div>

<!-- Remaining body text (What They Know, Treasure, etc.) continues after the
     stat block, as normal flowing content. -->
```
```

### NPC dialogue

```html
<blockquote class="say">
  <p><b>[NPC Name]:</b> &ldquo;[Line of dialogue.]&rdquo;</p>
  <p>[More dialogue as a separate paragraph if it's a distinct beat.]</p>
</blockquote>
```

### d6/d20 reaction or random tables

```html
<table class="dmtable">
  <tr><th>d6</th><th>Result</th></tr>
  <tr><td>1&ndash;2</td><td>[Result text]</td></tr>
</table>
```

---

## Render command (headless Chrome)

Both header-suppression flags are required together, or Chrome adds a default
timestamp/URL header and footer to every page:

```bash
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" \
  --headless --disable-gpu --no-sandbox \
  --print-to-pdf="[Output Title].pdf" \
  --print-to-pdf-no-header --no-pdf-header-footer \
  "file://[absolute path to index.html]"
```

## Verification loop

There is no `pdfinfo`/`pdftoppm`/`poppler` in this environment — do not try to install
them or shell out to Python for PDF rasterization. Instead:

1. Serve the build directory: `npx -y serve -l <port> .` (background; `pkill -f "serve -l <port>"` to stop it when done)
2. Use Playwright MCP: `browser_navigate` to the **http:// URL of the actual PDF** (e.g.
   `http://localhost:<port>/output.pdf`) — Playwright blocks `file://` entirely, so the
   static server is required even for local-only verification.
3. `browser_resize` to roughly `1100x1500`, then `browser_take_screenshot`.
4. Chrome's PDF viewer renders inside a shadow DOM — `document.querySelectorAll` /
   `scrollIntoView` via `browser_evaluate` do **not** reach into it. For checking a specific
   section's styling (not exact pagination), navigate to `index.html` directly instead and
   `scrollIntoView` the element by id/class — that's a good proxy for component styling,
   but only the actual PDF screenshot confirms real page breaks.
5. Read the screenshot file with the `Read` tool and visually check against the user's ask
   before reporting anything fixed.

## Known failure modes (already fixed in the CSS above — don't reintroduce them)

- **White border around full-bleed background**: caused by `@page margin` > 0. Chrome's
  print engine will not paint ANY background (not `html{}`, not `position:fixed`) into the
  page-margin area. Fix is `@page{margin:0}` + the `.pagewrap` box-decoration-break trick.
- **Giant whitespace gap before a full-width image inside two-column text**: caused by
  `column-span:all` forcing the preceding columns to balance height first. Fix is sibling
  blocks, not a spanning element (see `.mapfig` comment above).
- **Full-width image bumped to its own page**: caused by forcing `width:100%` on an image
  whose aspect ratio makes that width taller than the remaining page space. Fix is
  `max-height` instead of forced full width.
- **Title page overflowing onto a second near-blank page**: caused by title text wrapping
  to more lines than expected at a larger font size inside a fixed-height box. If the user
  asks for bigger body text, scale body/room text but keep the title page's own type sizes
  conservative, or widen `.titlepage` padding, and verify the title page still renders as
  exactly one page before moving on.
- **A dialogue quote splitting mid-sentence across a column/page break**: fix is
  `break-inside: avoid` on `blockquote.say p` (already in the CSS above).
- **A room's circular number badge getting sliced in half at a column break, with the
  header re-rendering fresh at the top of the next column**: `avoid-page` alone isn't enough
  — it only applies to page-break fragmentation, and Chrome treats column breaks as a
  separate context, so a flex element like `.area-head` (number badge + title) can still get
  cut mid-column. Fix is `break-inside: avoid` on `.area-head` specifically (already in the
  CSS above) — verify by screenshotting the actual page boundary where a room entry lands
  near a column split, not just the component in isolation.
- **Don't overcorrect the above by putting `break-inside: avoid` on the whole `.area` block**
  instead of just `.area-head`. That "fixes" the badge but is wrong: it forces the entire
  room entry (flavor box, encounter, body paragraphs, treasure box) to jump as one unit to
  the next column/page whenever the full entry doesn't fit the remaining space, leaving a
  large, visually obvious whitespace gap at the bottom of the column it left. Scope the
  break protection to the small header row only; `break-after: avoid` on `.area-head` keeps
  it glued to the first line of body text without dragging the whole room along. Reproduced
  and confirmed in a real build: the overcorrected version inflated the document from 23
  pages to 28 purely from whitespace gaps scattered through every room that happened to
  straddle a column boundary.
- **Stat blocks must never wrap, but can't just be `break-inside:avoid` and dropped wherever
  they fall**: with nothing else to fill the remaining column space first, forcing the whole
  (often tall) box to the next column/page leaves the same kind of large gap as the `.area`
  overcorrection above. Fix: put real body text (tactics, a trigger condition — whatever
  would narratively follow anyway) between the preceding content and the stat block in the
  HTML source, so the column fills naturally before the forced push happens. See "Stat block
  + portrait: placement pattern" above.
- **The same gap can appear one step earlier, before a full-width `.creature-art` portrait**:
  an `<img>` is not fragmentable at all (unlike text or even a stat block's constituent
  elements) — it either fully fits the remaining column space or it doesn't, no partial
  rendering. If a room's heading/flavor/encounter box happens to end partway down a column,
  the portrait (plus `break-inside:avoid` on its wrapper to keep the caption attached) gets
  pushed whole to the next column, leaving the same kind of gap. Fix is identical to the
  stat-block case: move some body text to appear BEFORE the `.creature-art` block too, not
  just before the `.statblock`. There's no way to know in advance how much text is "enough"
  — render, screenshot the actual page boundary, and adjust how much text precedes the image
  if a gap appears.
- **Generic type vs. unique individual naming**: a `.statblock`'s `<h4>` and a
  `.creature-art`'s caption should name the creature TYPE ("Reaper of Bhaal"), not a specific
  NPC instance's own name ("Lubash"), UNLESS that NPC is genuinely one-of-a-kind (a named
  boss like "Mortlock Vanthampur" — there's no separate "type" for him to fall back to). This
  keeps the art and stat block reusable if the same creature type appears again elsewhere in
  the document; the specific instance's name belongs only in the room's prose/encounter box
  text (e.g. "Lubash, a Reaper of Bhaal..."), optionally as an `a.xref` link to the shared
  stat block anchor.
