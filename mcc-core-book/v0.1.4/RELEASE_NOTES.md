Release v0.1.4

## Release Notes:
* mcc-core-text: rebuild Chapter 3 genome-focus table from source layout

The journal builder transposed this table (book lays it out as roll-range
rows x Mutant/Manimal/Plantient columns) incorrectly: it collapsed the two
failure rows into a single colspan, dropped the Mutant "surviving" entry, and
shifted the Manimal/Plantient cells several columns left -- so their 16-19..34+
results were mislabelled and the 32-33 / 34+ columns appeared missing.

Rebuilt to match the 6th-printing interior (pp. 114-115), verified cell-by-cell
via PyMuPDF find_tables():
- orientation restored to Roll | Mutant | Manimal | Plantient (4 cols, 10 rows)
- rolls 1 and 2-13 are the shared failure results (colspan=3)
- row 14-15 "surviving" filled for all three genotypes
- rolls 16-19 .. 34+ realigned to the correct genotype columns

All cell text already existed in the module except the Mutant "surviving" line
and the second failure row, both taken from the licensed source PDF.

Source JSON only; .ldb still needs `npm run todb` once Foundry releases the lock.

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com> (Tim White)
* mcc-core-text: repair 8 more tables with split/crammed columns

Same defect class as Table 10-1 — InDesign line breaks split cells or
crammed two columns into one, leaving ragged column counts:

- Table 1-1 (Ability Score Modifiers): merge stray "-1 program" / "*" split
- Table 1-4 (Additional Beginning Equipment): pad 2 continuation rows to 4 cols
- Table 1-6 (Mutant Appearance): rejoin mid-sentence split in d30 25-27 row
- Table 2-7 (Rover): add missing trailing cell to subheader row
- Table 2-13 (Plantient): split crammed "+6  95%" into Artifact/Hide columns
- Table 4-1 (Attack Roll Modifiers): colspan=2 on "Attack Roll Modifier" header
- Table 6-3 (AI Ability Ranges): rebuild as clean 3-col Score|Intelligence|Ego
- Table 8-1 (Conversing): colspan=3 on "With Data Ghosts" band header

Source JSON only; .ldb still needs `npm run todb` once Foundry releases the
pack lock. One uncaptioned Mutations grid (Manimal/Plantient rows missing the
32-33 and 34+ columns) is left for source verification.

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com> (Tim White)
* mcc-core-text: repair Table 10-1 (Base Artifact checks) column structure

The header crammed "Level" and "Artifact Check Bonus" into one cell, and
continuation rows (e.g. 5-8/+6) had 2 cells while named rows had 3, which
misaligned columns and flung the bonus values to the far-right edge.

- span the "For DCC RPG Characters" band across all columns (colspan=3)
- split into proper Class | Level | Artifact Check Bonus headers
- add an empty leading cell to every continuation row -> consistent 3 cols

Source JSON only; .ldb still needs `npm run todb` once Foundry releases the
pack lock (world v14 is currently running).

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com> (Tim White)
* Repair last U+2028 clip (Magnetic Control 30-31) + harden repair script

Follow-up to the RollTable audit Tim requested. Verified all 7 RollTable
packs (174 docs / 1779 rows): U+2028/tab/<br> counts are 0 (structured
results[], so no journal-style tab-collapse), and all 183 crit/fumble rows
end in terminal punctuation.

The sweep found one clip the first pass missed: mutation table Magnetic
Control [30-31] ("…objects/creatures, or"). recoverTail's 200-char window +
first-period cutoff couldn't span this row's long multi-clause tail.
Recovered the full cell text from the journal (authoritative intact copy,
prefix-checked) and hardened recoverTail: window 200->1500, stop at the
next cell's U+2028, and when the next row's range number is known capture
the whole remainder up to it instead of the first period.

Everything else flagged is a non-issue: colon stat-block lead-ins and short
stat/list lines lacking a trailing period in the source too (not content
loss). npm run check green (925 tests). LevelDB recompile pending.

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com> (Tim White)
* mcc-core-text: rebuild straggler mutation result tables + tab-block sweep

journal-fidelity-pass3.mjs (dry-run default, --apply; idempotent). Tim
flagged per-mutation d20 result tables rendering run-together, and asked
for a sweep of other table-like blocks.

1) spell-row -> spell-roll-table (11 tables). Most per-mutation result
   tables were already real <table class="spell-roll-table"> (84); a subset
   stayed as <p class="spell-row"> tab+<br> blobs. A mutation's table is
   split across consecutive spell-row <p>s, and a mid-cell U+2028 orphaned
   some row tails into their own <p> (…-4 Agility,</p><p>-2 Strength.</p>).
   The converter merges each consecutive run into one table, rejoining
   orphaned tails. Matches the existing 84 exactly (2-col tbody, no thead).
2) mutation-general leading label bolded (94): "General\t…" /
   "Manifestation\t…" tab collapse -> <strong>General</strong> ….
3) 4 artifact TL/Complexity stragglers pass 2 missed (varied <strong>
   placement) reflowed \t\t -> <br>.
4) Foreword "•\t" bullet gap -> "• ".

Prose-only spell-row blocks (Patron intros, Bestiary Manifestation; 9 left)
kept as paragraphs with tabs collapsed. Only tab-bearing <p> remaining is
the intentional deed-die credits block. npm run check green (925 tests).
LevelDB recompile pending (Foundry lock; JSON src canonical).

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com> (Tim White)
* Repair U+2028 mid-sentence clips + strip remaining dead journal attrs

Two follow-ups from journal review:

1) repair-u2028-clips.mjs — a reviewer flagged two Crit Table I entries
   (rolls 9, 11) truncated mid-sentence. Root cause: a U+2028 (Unicode
   LINE SEPARATOR) mid-cell in the source; walk-stories converts it to
   <br>/space for the journal (intact there), but the tab-segment table
   parsers truncated each affected cell AT the U+2028, dropping the rest.
   Audit of all 283 source U+2028 + a precise shipped-data scan found 18
   genuinely clipped RollTable fields (5 crit/fumble + 13 per-mutation).
   The script recovers the authoritative tail from the raw IDML Story XML;
   rows with identical prefixes are disambiguated by the next row's range
   number (Gene Splice correctly resolves to 1d3/1d4/1d6/1d20). Idempotent;
   journal copies confirmed intact; artifact stat blocks retain every line.

2) journal-fidelity-pass2.mjs — extended with fixes 7-8 after an attribute
   census: strip the leaked IDML data-self id (1) + the only raw <a>
   (an empty, invalid-id leaked anchor), and normalize the lone
   class="real-table" to stat-table so it picks up shared table styling.

npm run check green (925 tests). LevelDB recompile pending (Foundry lock;
JSON src is canonical).

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com> (Tim White)
* mcc-core-text: journal fidelity pass 2 (tables, headers, idml-style strip)

A fresh structural re-scan of all 18 journal pages surfaced six issues past
the 2026-05-30 batch. One consolidated idempotent script
(scripts/journal-fidelity-pass2.mjs, dry-run default + --apply):

1. AI Personality generator (Table 6-4) rebuilt — 16 trait rows were
   stranded in a tab-delimited <p> and "(once for each column)" was torn
   across a broken tbody row + a stray bold fragment. Now a real 4x16 table
   with the two sub-notes reattached to their columns.
2. Combat Activity/Time table rebuilt from a flattened tab+<br> <p>
   (+ a stranded 9th row), with the * footnote lifted to <p.table-footnote>.
3. 5 Patron Taint tables: floating bold "Roll | Patron Taint Result"
   headers lifted into real <thead>.
4. 52 artifact stat lines: "Tech Level: N \t\t Complexity Modifier: N"
   (tabs collapse in HTML) reflowed onto their own <br> lines.
5. Heading hygiene: Foreword "HOPELESS CHARACTERS" -> <h2>; the 5 crit-table
   cross-link <h4>s under "Critical Hits" demoted to <h3> (fixes h2->h4 jump).
6. Stripped 1549 data-idml-style provenance attrs — nothing consumes them
   (no CSS selector, no module JS); ~46KB of dead weight in shipped HTML.

npm run check green (lint + scss + 925 tests). LevelDB recompile pending
(Foundry holds the lock; JSON src is canonical).

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com> (Tim White)
* mcc-core-text: remove stray Chapter-4 divider from Mutations page

Traced via the IDML: the "Chapter Four / COMBAT" chapter-title was
paragraph 133/134 of story ue69d (mental mutations, book pp.74-75) --
InDesign threaded the Chapter-4 title frame onto the end of the Mutations
content. The real Combat chapter is a separate story (uf17e, p.120,
opening "Overview"), so in a per-chapter Foundry journal this divider is
noise. scripts/fix-stray-chapter-divider.mjs removes it and joins the
internal <br> in the standalone u35dc "...peasants and serfs..." callout
that shares the spread (kept -- it's real book content). Confirmed
isolated: no other stray chapter dividers. 925 tests green.

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com> (Tim White)
* tables: extend inline rolls to all RollTable packs

Run add-inline-rolls.mjs across mutation / AI-program / artifact / misc
tables / disapproval / glowburn packs: 565 dice wrapped in [[/roll NdM]].

Fix add-inline-rolls.mjs to protect @UUID/@Compendium links and HTML tags
(not just existing [[ ]] rolls) — a dice regex match was wrapping an
all-digit "...d..." run inside a @UUID target ID (Item.97595358786839d4),
breaking the reference. Now splits on protected tokens via index parity.
925 tests green (uuid-references no longer broken).

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com> (Tim White)
* crits-and-fumbles: wrap result dice in inline rolls

scripts/add-inline-rolls.mjs [pack] wraps bare dice expressions in
RollTable result descriptions with [[/roll NdM]] so they're clickable
(matches dcc-core-book's /roll convention). Leading sign stays outside
(+[[/roll 1d8]]), trailing modifier stays inside ([[/roll 1d4+1]]);
idempotent (skips dice already inside [[ ]]). 142 dice across the 8
mcc-core-crits-and-fumbles tables. Pack-parameterized for later reuse on
the other table packs / journal display tables.

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com> (Tim White)
* mcc-core-text: journal fidelity batch (notes, dropcap, tables, headings)

Six idempotent JSON-level patches on the journal source (IDML builders
frozen), closing §3 fidelity items 3,4,6,7,8,9:

- delete-editorial-notes.mjs: remove 2 leaked "...will send when ready"
  production placeholder paragraphs.
- restore-dropped-initials.mjs: restore the lone dropped initial
  ("he mutant species" -> "The ..."). The other 3 lowercase-starts were a
  Spell-Table row + 2 adventure credit lines (correctly lowercase).
- rebuild-title-tables.mjs: convert all 7 class "Titles" tables (tab-
  delimited <p> runs + a caption stranded as an orphan row in the prior
  progression table) into real <table> with caption + Level|Title thead.
- split-uuid-headings.mjs: de-absorb 14 @UUID headings -- 10 Artifacts
  category headers split (heading + item <p>), 3 compound titles <br>->
  space (Combat crit table, 2 Bestiary creatures); 1 Mutations
  chapter-title skipped + flagged (mis-extracted).
- lift-table-footnotes.mjs: move 5 *,**,*** footnote rows out of the
  Character Creation tables into <p class="table-footnote">.
- add-table-subheader-colspan.mjs: colspan + class on 6 bold group
  sub-header rows so they span multi-column tables.

styles: .table-subheader (span/shade) and p.table-footnote (small) CSS.
925 tests green.

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com> (Tim White)
* styles: space stacked stat tables in MCC journals

Crit/fumble and other stat tables stack back-to-back in the journal (each
table's <caption> carries the next table's title / rollable-table link),
so with no explicit spacing they read as one continuous grid. Add
margin-block to table.stat-table (scoped to .mcc-core-book content) plus a
small caption gap. Measured ~66-150px separation between consecutive
tables in the live preview.

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com> (Tim White)
* mcc-core-text: heading + table fidelity passes (promote, unbreak, merge)

Three JSON-level patches on the journal source (IDML builders frozen),
each idempotent with a dry-run/preview workflow:

- promote-bold-headings.mjs: 8 standalone bold-only <p>s that are real
  sub-heads ("The Setting: Terra A.D.", "What Is This?", ...) -> <h2>.
  Filter excludes table-header rows, statblock lines, over-bolded
  sentences, the byline, and the MCC# adventure list.
- fix-heading-line-breaks.mjs: 22 heading-internal <br> -> space, so
  titles flow/wrap instead of hard-breaking ("Critical Hits by
  Monsters"); also repairs the "missing space" artifact those breaks
  caused ("Rules Mechanics", "Climate and Ecology"). Skips the 14 @UUID
  headings (a separate "absorbed content" fix). Open tag captured by
  regex group so heading attrs (class/data-idml-style) are preserved.
- merge-table-continuation-rows.mjs: merge 5 orphan single-cell crit-table
  rows back into the previous Result cell (e.g. "...streams down the" +
  "enemy's face."). Conservative filter (orphan starts lowercase/digit,
  prev cell not sentence-terminated) leaves footnotes, captions, and
  group sub-headers untouched. Positional surgery preserves thead/tbody
  and row classes.

normalize-journal-headings.mjs: canonicalize dotted initialisms
("a.d." -> "A.D."); period-less tokens (AIs/PCs) untouched. Guard green.

docs/00-progress.md: close item 5; log remaining journal-fidelity items
(drop caps, leaked art-notes, @UUID headings, footnote rows, title
tables, group-header colspan).

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com> (Tim White)
* mcc-core-text: split <br>-joined prose into separate paragraphs

InDesign fused some prose paragraphs with <br>, which defeats the CSS
paragraph-spacing flow in journals. scripts/split-br-paragraphs.mjs
(idempotent, JSON-level patch — IDML builders are frozen) splits the 185
prose <p>s on <br> into sibling <p>s, KEEPS the 33 structured ones
(statblock / credits / index short-line stacks), and LEAVES + reports 70
"mixed" for manual review.

Conservative classifier: prose = a <p> with >=1 long sentence-like
segment and <=1 short line; structured = mostly <=60-char lines.
<p> 849->1267, <br> ~1223->798. No loose (un-wrapped) text exists in the
journal. Validated live via an injected world preview journal.

Compile deferred: batching with the §3 journal-fidelity passes (leaked
art-notes, dropped drop-caps, bold-as-heading, class title tables) so the
LevelDB lock is only broken once.

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com> (Tim White)
* Upper-level pregens: disable showSpells on Mutant/Manimal/Plantient

These three classes render their own "Mutations" tab (part id `spells`).
With config.showSpells=true the DCC base sheet also injected its generic
"Spells" (wizardSpells) tab, producing a duplicate tab with identical
content. The class defaults intentionally leave showSpells off (schema
default false); the pregens had it set true. Flip to false to match.
Shaman keeps showSpells=true (genuine caster, no mutations tab).

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com> (Tim White)
* Add scene-thumbnail normalizer; run it after tojson

Foundry regenerates each scene's thumbnail into the active world folder
(worlds/<world>/assets/scenes/thumb-*.webp) on every scene save, so a
round-tripped adventure keeps reverting `thumb` to a world-local path that
fails test/img-paths. scripts/normalize-scene-thumbs.mjs rewrites each
scene's thumb to a stable module-local thumbnail derived from its v14 Level
background (scene.levels[].background.src), generating the thumb from the
background via sharp when missing. Idempotent; accepts optional pack-name
args. Wired into `npm run tojson`; also `npm run fix-thumbs`.

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com> (Tim White)
* Salvage Pits: link Bubble-Car table from Area 2-3; re-fix scene thumbs

Tim added the @UUID[RollTable.b7a6b636cfc2d5ae] link on the Area 2-3
(Tarpaulin-Covered Pit) page in Foundry. All three scene thumbnails again
reverted to world-local paths on scene save; repointed to module thumbs.

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com> (Tim White)
* Salvage Pits: add Rooje Camp scene + art pages; fix scene thumbs to module paths

Round-trips Tim's in-Foundry adventure edits via extractPack:
- new "Rooje Camp" scene backed by assets/maps/rooje-camp-detail-map.webp
- five new journal art/map pages (Jobber's Camp Map, New Generator Art,
  Overview Side Map, Rusty Room Rescue Art, The Lost Bunker Map)
- journal links to the 5-3 Supply Room and 5-4 Generator Malfunction tables
- table renamed "NEW Generator Malfunction Effects" -> "Generator
  Malfunction Effects (Area 5-4)" (same _id)

Fix: all three scenes' thumbnails had reverted to world-local paths
(worlds/mcc/assets/scenes/thumb-*.webp) on scene save. Repointed Rooje
Hill + Lost Bunker to existing module thumbs and generated a new
assets/maps/thumbs/rooje-camp-detail-map.webp (300x277) for Rooje Camp.

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com> (Tim White)
* Salvage Pits: extract Areas 5-3/5-4 RollTables, drop orphan viewscreen table

The 5-3 Supply Room (1d8) and 5-4 NEW Generator Malfunction (1d6) tables
existed only as static HTML in the journal pages; extracted both into
RollTables embedded in the adventure's tables[] (RollTable folder
HqEEGT90P8NmYtMl), preserving inline [[/r ...]] rolls. Foundry round-trips
these inside the single Adventure document, so no separate src file.

Removed the "Office AI Workstation Viewscreens" table (1d16): confirmed
via the IDML source (Story_u1116c.xml) to be an orphan frame parked on the
Salvage Pits print spread but containing a foreign A-/C- area complex
(Tube-Thing, Garden of Wee-DN, Po-Z) absent from the Salvage Pits text
(1-1..5-4 only) and from Weirding Music. Likely a leftover from an earlier
printing; not core-book content.

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com> (Tim White)
* Adopt Foundry title-based pack filenames; freeze extractPacks workflow

One-time reconciliation after a Foundry editing session + `tojson` run.
extractPacks names files by document title while the original build-*.mjs
scripts named them by slug, so re-extracting recreated every doc under a
new filename (duplicate _id collisions). Keeping Foundry's title-based
names: removed the 85 old slug-named files, staged the title-named ones
(84 detected as renames). Applied Tim's live adventure edits to the two
adventure scenes + rooje-hill-surface.webp; the 57 in-place mods are
Foundry coreVersion "14" -> "14.363" stamp churn.

Repointed two Salvage Pits scene thumbs that Foundry had rewritten to
world-local paths (worlds/mcc/assets/scenes/thumb-*.webp) back to the
existing module thumbs under assets/maps/thumbs/ (caught by img-paths
test).

Workflow rule going forward: do NOT run tojson/extractPacks. The pack
src JSON is the source of truth, edited directly or by Tim in Foundry,
hand-reconciling only touched files. Noted in docs/00-progress.md.

Adds bolter-gun.svg (slug thrower rifle icon).

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com> (Tim White)
* Fix blank adventure scenes: restore Level background on import

v14 moves a Scene's map onto a child Level (scene.levels[].background.src), and
that src is a *virtual* FilePathField. When the Adventure compendia are loaded,
Foundry's package-data migration drops the virtual src from the bundled Scenes
(confirmed live: a Scene in a plain Scene pack keeps it; the same Scene bundled
in an Adventure comes back with src:null), so imported maps rendered blank.

The map path survives in each Scene's mcc-core-book.sourceImage flag, so add a
preCreateScene hook that re-applies it to the first Level when the background is
missing. Verified end-to-end in Foundry v14.363: deleting + re-importing the
Salvage Pits adventure now recreates both scenes with their maps intact. Also
covers a judge dragging a Scene straight out of the compendium.

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com> (Tim White)
* Salvage Pits journal: restore original room-numbered maps

The journal map figures (assets/decoration/under-the-salvage-pits.webp,
rooje-hill.webp) had been overwritten with the rotated, number-stripped scene
derivatives during the battlemap-prep work. Regenerate both from the pristine
.source/Links originals (768px/q80, native orientation) so the journal shows the
printed maps WITH their area numbers (4-1..5-4, 1-1..3-2) for cross-referencing
the encounter text. The play battlemaps under assets/maps/ stay rotated/cleaned.

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com> (Tim White)
* Adventure scenes: emit native v14 Level schema (lossless round-trip)

Scenes were built in the pre-v14 schema (top-level scene.background), which
Foundry migrates onto a child Level document on import and then drops the
top-level key on export — so every align-in-Foundry round-trip looked like it
'lost' the background and offsets.

Emit the native v14 shape from the start: the map image lives on a default
Level (scene.levels[].background.src) with initialLevel set, grid alignment in
scene.shiftX/shiftY, and the modern fog/environment/transition fields. Bake the
measured Salvage Pits alignments (Lost Bunker shift -17/-16, Rooje Hill -11/-15,
grid.alpha 0 to hide the redundant overlay over each map's printed grid) so they
import pre-aligned. Drop the dead 'templates' collection (not in the v14 Scene
schema). Update the Scene doc-schema test to assert levels[].background.src.

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com> (Tim White)
* Adventure scenes: rescale Rooje Hill to 35px grid + generate map thumbnails

- Rooje Hill (Surface) grid measured at ~34.5px; rescaled the high-res source
  to an exact 35px grid (verified 34.98/35.01), scene now 2076x1035 with
  grid.size=35.
- Generate a ~300px Scenes-directory thumbnail per adventure map into
  assets/maps/thumbs/ and set scene.thumb, so scenes show a preview on import
  instead of thumb:null. walkAssetPaths now validates `thumb` paths too.

925 tests green.

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com> (Tim White)
* Lost Bunker map: rescale to an exact 44px grid + bake grid.size=44

Measured the printed grid pitch via FFT (~43.5px, slightly anisotropic) and
rescaled the high-res source ~1% non-uniformly so the grid is exactly 44px in
both axes (verified 44.00/44.00). Scene now ships grid.size=44 at 2036x2067, so
on import the GM only nudges the grid offset, not the size. Regenerated the
matching 768px journal figure.

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com> (Tim White)
* Lost Bunker map: second rotation pass (resize scene to 2006x2048)

Re-rotated map re-converted to the 2048px scene map + 768px figure; the new
image is 2006x2048 (was 1577x2048), so the Lost Bunker scene's width/height
were refreshed and the adventure re-bundled.

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com> (Tim White)
* Lost Bunker map: replace with grid-straightened rotation

Tim rotated "Under Rooje Hill" / The Lost Bunker so its printed grid is
axis-aligned. Re-converted the edited PNG to the 2048px scene map
(assets/maps/) and the 768px journal-figure copy (assets/decoration/).
Dimensions unchanged (1577x2048), so the scene doc/LevelDB is unaffected —
the scene references the file by path.

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com> (Tim White)
* Adventures: paragraphs, inline dice rolls, #N lists, table fix, drop roster

Batch of journal-formatting fixes in the assembler:
- Narrative/section text now paragraph-formats (a single <br> at an inline-tag
  boundary, e.g. </em><br><em>, is a paragraph break, not a line wrap), so
  Player Start etc. are real <p> paragraphs instead of one wall.
- Dice expressions ("1d6", "2d5+6") wrap as inline rolls [[/r 1d6]] (skipping
  tags, @UUID targets, existing rolls) — 93 across the two adventures.
- Run-in "#1: … #2: …" gear lists become <ol><li> (joins <br>/</p><p> seams
  between items; trailing prose cut off the last item).
- The transposed PC's Intent lookup table (2×6) is un-rotated to 6×2.
- Dropped the redundant "Creatures & Tables" roster page (NPCs are linked
  inline + foldered; tables are embedded).

925 tests green; all pages balanced.

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com> (Tim White)
* Strip leaked InDesign <?ACE n?> processing-instruction markers

walk-stories captured InDesign PIs (<?ACE 7?> auto-number markers, <?ACE 3?>,
<?ACE 18?>) as run text; build-html HTML-escaped them, leaving literal
"&lt;?ACE n?&gt;" junk inside journal lists/text.

- build-html escapeHtml now drops <?…?> PIs at the source (future builds clean).
- build-adventures strips residual escaped/raw markers per page on rebuild.
- scripts/strip-indesign-markers.mjs (npm: strip-indesign-markers) scrubs the
  already-shipped pack JSON without a lossy rebuild; removed 28 markers from
  mcc-core-text. Idempotent.

No markers remain in any pack. 925 tests green.

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com> (Tim White)
* Adventures: format every area page (blockquote read-aloud + judge paragraphs)

Each area page now drops the "Area N-M — Name:" lead (it's the page title), puts
the leading read-aloud italic text in a <blockquote>, and wraps the judge text
in <p> paragraphs (2+ <br> -> paragraph break, single <br> -> space). NPC
enrichment runs before the area split so stat-blocks are already their own
linked paragraphs, which the area formatter preserves verbatim. Applies to all
areas in both adventures.

925 tests green.

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com> (Tim White)
* Adventures: catch inert/object NPCs, link + space NPC stat-blocks

- Broaden the NPC detector to non-numeric Init (N/A / none / last) with an Atk
  guard, adding Lab Tech Robot Body and Iron-imbued skeletons (16 NPCs now). The
  guard's first cut accidentally required \b after Init and dropped Corliss
  (Init 1d3+2, a die expression); fixed by removing the \b.
- assign-icons: robot-golem (Lab Tech Robot Body), skeleton (Iron-imbued
  skeletons); all 16 NPCs iconed.
- build-adventures: enrichNpcBlocks lifts each inline "<strong>Name:</strong>
  Init …" stat-block onto its own paragraph (so it's set off from the prose) and
  links the bold name to its bundled Actor via @UUID[Actor.<id>].

925 tests green.

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com> (Tim White)
* Adventures: catch all keyed areas + nest area pages as level 2

The area splitter missed headers led by empty <a name></a> bookmark anchors
(2-1, 2-2, 3-1, 4-2) and used too-narrow lead/separator matching. Fixed: a real
area header is "Area N-M – Name:" (capital, colon-terminated, not inside an <a>
link), led by <br>, a <p> open, or a bookmark anchor; cross-references (ending
in "." or flowing into prose) are excluded by the colon.

Result: Weirding Observatory 1-1..1-11 (11 pages); Salvage 1-1..5-2 (17 pages),
sequential within sections. Area pages are now title.level 2 so they tuck under
their parent section in the journal nav. HTML stays balanced; no content lost.

925 tests green.

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com> (Tim White)
* Adventures: file the journal under its own folder on import

Each Adventure bundle now includes a JournalEntry folder named after the
adventure, and the single journal is filed under it — matching the Actor and
Scene folders, so an import lands the journal in a named folder rather than at
the world root. Applies to both adventures.

925 tests green.

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com> (Tim White)
* Adventures: per-area journal pages + capture in-Foundry scene edit

- build-adventures.mjs: split each keyed "Area N-M" entry into its own journal
  page (draggable onto the map as a map note), named "Area N-M — <name>".
  Applied in the assembler so the enrichment cross-links in the journal
  intermediates survive. Salvage Pits → 7 area pages; Weirding → 1 (the source
  only formally keys areas where a paragraph starts with "Area N-M").
- Captured the in-Foundry edit on Xocust Mountain — Lab Level (39 walls + 4
  tokens, tokens correctly referencing the bundled Prokyonic Raiders actor) by
  folding the authored layers into the scene intermediate, keeping the builder's
  background/grid.
- build-adventure-scenes.mjs now preserves GM-authored placeable layers
  (walls/tokens/lights/notes/...) on rebuild, so future regen can't wipe map work.

925 tests green.

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com> (Tim White)
* Adventure NPCs: assign relevant game-icons.net icons

The 14 appendix-adventure NPCs were on the mystery-man placeholder. Extend
assign-icons.mjs to cover the build intermediate (metadata/adventure-build/
creatures/, via a new SRC_DIRS override; marked PARTIAL so its folder docs are
skipped) with a statblock-fitting icon each:

  Lab Tech Bot Arm->robot-grab  Macknumsianas->tentacle-strike
  Prokyonic Raiders->raccoon-head  Barkt->wolf-howl  Changling->transform
  Corliss->brain  Cydog->wolf-head  Gang member/Jobber's gang->robber-mask
  Jobber->spiked-bat  Lightscreech->bat  Ruffiz->rat
  Scuttle Flower->carnivorous-plant  Smart mud->slime

9 new SVGs vendored. Reassembled both Adventures so the bundled actors carry
the icons (img + prototypeToken.texture.src). 925 tests green.

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com> (Tim White)
* Weirding Music: judge's note on isometric/gridless maps

Append a note to the end of the adventure journal's Introduction page: the
Xocust Mountain maps ship gridless (isometric projection), with a pointer to the
Isometric Perspective module for judges who want true iso grids. Injected by the
assembler (build-adventures.mjs) so the enrichment cross-links survive.

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com> (Tim White)
* Adventure scenes: isometric Xocust maps ship gridless

The three Xocust Mountain maps are drawn in isometric (oblique) projection —
rhombus floor grid, diamond "5 ft" legend. Core Foundry has no isometric grid
type, so a square overlay can't align (rotating doesn't help) and the only true
fix is the Isometric Perspective module (an unwanted dependency). Ship them
GRIDLESS (CONST.GRID_TYPES 0): tokens place freely, the ruler measures via
grid.distance (5 ft). The two Salvage Pits maps are genuine top-down and keep a
square grid. Rebuilt the scene intermediates + reassembled both Adventures.

925 tests green. LevelDB recompile pending (Foundry running; JSON src canonical).

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com> (Tim White)
* Adventures: repackage as two Adventure-type compendia (one per adventure)

Per Tim: each adventure should be its own compendium with its roll tables
embedded. A Foundry compendium is single-type, so "one compendium = one whole
adventure" is realized via the Adventure document — the native one-click-import
bundle. Replace the three by-type packs (JournalEntry/Actor/Scene) with two
Adventure compendia:

  mcc-core-adventure-weirding-music   (1 Adventure: 10-page journal, 3 NPCs, 3 scenes)
  mcc-core-adventure-salvage-pits     (1 Adventure: 14-page journal, 11 NPCs, 2 scenes, 2 tables)

- scripts/build-adventures.mjs assembles the Adventure docs from build
  intermediates (the former by-type pack src, git-moved to
  metadata/adventure-build/; the component builders now write there). Bundled
  cross-links are rewritten to world-relative @UUID[Scene/Actor/RollTable.<id>]
  so they resolve after import; core-pack links (artifacts, Table 7-1, text)
  stay Compendium UUIDs. `_key` stripped from embedded docs.
- The 2 roll tables move out of mcc-core-tables (back to 16) into the Salvage
  Pits Adventure.
- Content importer: new "Adventures" group; handler calls Adventure#import
  (unpack into world) instead of importAll for Adventure packs.
- Tests learn the Adventure type (module-manifest, pack-keys, doc-schema).

925 tests green; both Adventure packs compile to LevelDB. Needs an in-Foundry
import check by Tim (Adventure import can't be verified headless).

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com> (Tim White)
* Appendix adventures (4/4): journals + move content out of core-text

Final phase: build the two adventure JournalEntries and lift their content out
of the core-book text journal.

- packs/mcc-core-adventures (new JournalEntry pack) via
  scripts/build-adventure-journals.mjs: one JournalEntry per adventure
  (Weirding Music 10 pages, Salvage Pits 14 pages), paginated by section, with
  maps/art as inline figures. A "Maps & Scenes" page links each battlemap →
  its Scene; a "Creatures & Tables" page rosters @UUID links to the 14 NPC
  Actors + the 2 roll tables. Prose cross-links into core packs added by the
  UUID rewriter.
- Content move: surgically removed the "Appendices" page + its ToC entry from
  mcc-core-text (preserving the rest of its links — no lossy rebuild), and
  taught build-journals.mjs to skip backmatter-appendices via EXCLUDED_CHAPTERS.
- Wired the 3 adventure packs into module.json's MCC Adventures folder, the
  content importer (new Scenes group), and lang.

939 tests green. LevelDB recompile pending (Foundry lock; JSON src canonical).
Closes the appendix-adventures feature (scenes + tables + NPCs + journals).

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com> (Tim White)
* Appendix adventures (3/n): NPC Actor pack (foldered per adventure)

Add mcc-core-adventure-creatures: 14 tokenable NPC Actors from the two
appendix adventures (3 from Weirding Music, 11 from Salvage Pits), foldered
per adventure.

- scripts/lib/creature-statblock.mjs: shared stat-block grammar + DCC NPC
  builders, lifted from parse-creatures.mjs so the adventure parser reuses the
  exact Init/Atk/AC/HD/MV/Act/SP/SV/AL parsing. parse-creatures.mjs is left
  untouched (it emptyDir()s shared pack dirs and can't be safely re-run); a
  future cleanup can migrate it onto this lib.
- scripts/parse-adventure-creatures.mjs: per-line stat-block extraction (two
  blocks can share a paragraph, e.g. Jobber + Barkt), encounter-count stripped
  to a flag, explicit "hp N" folded in for token-bar fidelity, robotic NPCs
  (Lab Tech Bot Arm) routed to crit table A.
- module.json: declare the Actor pack under the MCC Adventures folder.

929 tests green. Stats reuse the bestiary grammar; verbatim stat-lines are
preserved in each NPC's notes. LevelDB recompile pending (Foundry lock).

Journals + content-move from mcc-core-text are the final phase.

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com> (Tim White)
* Appendix adventures (2/n): the two adventure roll tables

Add the genuine roll tables from "Under the Salvage Pits" to mcc-core-tables
via scripts/parse-adventure-tables.mjs:

- Office AI Workstation Viewscreens (1d16, 16 rows)
- Bubble-Car Failed Artifact Check Results (1d20 bands: 1 / 2-8 / 9-11 / 12+);
  its "Table 7-1" reference auto-links via the UUID rewriter.

Note: the "Phenomenometer" — sometimes counted as a third appendix table — is
actually an artifact stat-block in prose, not a roll table, so it stays in the
Weirding Music journal (built in a later phase), not here.

920 tests green. LevelDB recompile pending (Foundry lock; JSON src canonical).

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com> (Tim White)
* Appendix adventures (1/n): full-res maps + Scene pack

Begin packaging the two bundled appendix adventures. This phase ships the
battlemaps as Foundry Scenes.

- assets/maps/: re-converted 5 battlemaps from .source at 2048px/q85 (the
  shipped assets/decoration copies are 768px — too small for token maps).
  The 3 Xocust Mountain levels + Salvage Pits "Lost Bunker" + Rooje Hill
  surface. The new-generator / elevation / room art stay illustrations.
- packs/mcc-core-adventure-scenes (new Scene pack, 2 folders + 5 scenes) via
  scripts/build-adventure-scenes.mjs. Square grid at the book's scale (5 ft
  dungeon, 20 ft surface); grid.size is a placeholder — the GM aligns each
  scene to its printed grid in Foundry (pixel pitch can't be inferred here).
- module.json: declare the Scene pack under a new "MCC Adventures" packFolder.
- tests: teach the suite the Scene type (pack-keys COLLECTION, doc-schema
  width/height/background.src rule, module-manifest KNOWN_TYPES) and extend
  walkAssetPaths to validate Scene background/foreground src. 912 tests green.

LevelDB recompile pending (Foundry holds the lock; JSON src is canonical).
Journals + adventure NPCs + content-move from mcc-core-text follow.

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com> (Tim White)
* §4: close secondary journal packs as redundant (Tim's call)

mcc-core-journals / -creatures-journals / -mutations-journals /
-artifacts-journals would ship a second copy of prose that already lives in
the mcc-core-text chapters AND the per-entity Item/Actor docs, all of which
are already @UUID-cross-linked (711 links from the journal into those packs).
Decline to build them; parallels the collapsed weapons/armor/equipment/
ammunition packs and the non-viable mcc-core-mutation-effects. A lean
link-only catalog is noted as the cheapest future revisit if ever needed.

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com> (Tim White)
* chore: gitignore Claude Code runtime lock in .claude/

scheduled_tasks.lock is a transient harness lock (sessionId/pid/timestamps),
machine- and session-specific — never belongs in VCS. Add a nested
.claude/.gitignore (mirrors the .idea/ pattern) ignoring *.lock and the
per-developer settings.local.json, while keeping the shared settings.json
tracked.

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com> (Tim White)
* docs: compress §5 pregen-routing to archive (doc-hygiene cycling)

The §2 license-sweep commit (faa5139) added a new full-detail [x] item,
shifting the recent-5 window by one. Compress the now-6th-most-recent
discoveries-bearing phase-close item — §5 "Pregens — MCC class sheet
routing" (2026-05-27) — to a one-liner + archive pointer, and append its
full detail to docs/00-progress-archive.md under §5 (append-only, ordering
preserved between main and archive).

(The §5 broken-link audit wrap-up — code + its own doc-hygiene cycling of
the denylist item — already landed in b9ec767; nothing further was due there.)

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com> (Tim White)
* §2: license sweep — exclude third-party + Goodman product ads, scrub coupon

Tim's decision (2026-05-29): exclude Tier 1 (6 third-party publisher ads)
and Tier 2 (19 Goodman own-product ads + ad-spread background texture),
retain Tier 3 (core MCC/Goodman branding), and scrub the DriveThruRPG coupon.

- metadata/license-excluded.json: single exclusion source (images + stories).
- build-journals.mjs: filters excluded image anchors out of the gallery and
  skips the coupon story (u14aa5).
- convert-assets.mjs: skips excluded source images so a re-convert won't
  resurrect them.
- scrub-license-excluded.mjs (npm run scrub-license-excluded): idempotent
  surgical patch of the already-built journal. A full pipeline rebuild was
  avoided because build → rewrite-uuid → normalize-headings drops 2 hand-added
  cross-links; the scrub edits the journal HTML directly instead.
- 25 excluded WebPs deleted from assets/page-elements/.

Result: 25 gallery figures (8 Index + 17 Back Matter) + 1 coupon paragraph
removed; all 711 @UUID cross-links preserved; npm run check green (903 tests).
LevelDB recompile pending (Foundry holds the lock; JSON src is canonical).

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com> (Tim White)
* §5: broken-link audit — runtime GM tool + static auditor

Port dcc-core-book's checkForBrokenLinks as two complementary tools:

- module/checkForBrokenLinks.js: runtime GM tool, wired into the ready hook
  as game.mccCoreBook.checkForBrokenLinks. Resolves every @UUID across this
  module's packs live via fromUuid (so it validates external/world links too)
  and reports via DialogV2 + console.table.
- scripts/check-broken-links.mjs (npm: check-broken-links): static auditor over
  packs/*/src/*.json, bucketing refs into intra-module (resolved, incl.
  JournalEntryPage sub-targets), external, and relative/world; non-zero exit
  only on a broken intra-module ref.

Audit result: 0 broken / 0 external / 0 relative across 1616 references in
17 packs — every cross-link is internal and resolves.

Doc hygiene: flip §5 Broken-link audit to [x]; compress the §5 UUID-rewriter
plain-text-field denylist (2026-05-27) to the archive.

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com> (Tim White)
* docs: compress §9.1 to archive (doc-hygiene cycling)

The §9.1 quick-wins batch (single commit a0552c3) is now older than the
recent-5 [x] window (9.3a, 9.2e, 9.2d, 9.2c, 9.2b), so per the CLAUDE.md
hygiene protocol its full detail moves to the archive's new §9 block
(placed before 9.2a to preserve numeric order) and the four main-doc
items compress to one-line summaries with — see archive pointers.

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com> (Tim White)
* §9.3a: Naturopathy value-format refactor (parser, level-data, validator)

- parse-class-level-data: new parseNaturopathy() splits the book table's
  "1d3 (x2)" cell into .die/.usesPerDay/.usesRemaining; regenerated the
  10 Healer level-items (kept only real content changes, reverted the
  timestamp-only churn the parser stamps via Date.now()).
- Validation pipeline: naturopathy leaves skillMap (no longer a single
  .value); inventory captures the Healer sheet's NATUROPATHY_DICE ladder;
  new "Naturopathy per-day pool vs level-data (§9.3a)" group (5 green rows)
  verifies L1 die/usesPerDay/usesRemaining and that the sheet ladder tracks
  level-data die L1-L10 with usesPerDay == 2x level. Report Naturopathy
  section + Rollable-surfaces rows rewritten to "landed"; the stray
  "Table 2-5" reference corrected to 2-3 (the book cross-reference).
- Verdicts: 107 OK / 10 warn / 0 fail / 0 dash (was 101/11/0/0).
- Progress doc: 9.3a -> [x] with full discoveries; verdict + status lines
  updated; doc-hygiene cycling (9.2a compressed to the new §9 archive block).

npm run check green (lint + scss + 903 tests).

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com> (Tim White)
* Fix pregen skill labels: use canonical i18n keys (not invented MCC.Skill*)

The upper-level pregen parser baked skill labels as MCC.SkillArtifactCheck /
MCC.SkillAIRecognition / etc. — keys that don't exist in mcc-classes lang. The
skill-check chat flavor localizes skill.label, so imported pregens showed the
raw key ("rolled 15 for their MCC.SkillArtifactCheck check"). The schema/sheets
use MCC.ArtifactCheck / MCC.AIRecognition / Healer.Naturopathy /
Rover.DoorsAndSecurity / Plantient.HideInGreenery, so fresh actors were fine —
only baked pregens carried the bad label.

- parse-pregens-upper-level.mjs: use the canonical keys; drop the inert/incorrect
  `ability` on aiRecognition (book Ch.6: no ability mod) / naturopathy /
  doorsAndSecurity (no schema slot anyway). artifactCheck keeps ability:'int'.
- Patched the 7 baked upper-level pregens to match (label remap + ability align).
  Level-zero pregens were unaffected (no MCC skills baked).

LevelDB recompile pending (Foundry lock); re-import pregens to pick up the labels.

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com> (Tim White)
