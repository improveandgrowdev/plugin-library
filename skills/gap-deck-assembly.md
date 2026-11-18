---
name: gap-deck-assembly
description: 'Assembles the finished slides of an Improve & Grow (I&G) Growth Action Plan (GAP) into one deck in the standard order: cover, agenda, four numbered section dividers, a divider per revenue stream, Next Steps, Questions, and the I&G appendix (I&G Overview, Our Approach, Camps We''ve Served, Black Rock success story). Use whenever the user wants to put the GAP deck together, combine the slide files from the other gap skills, check which slides are still missing, set the slide order, build the agenda and section breaks, or add the standard I&G slides, even if they only say "assemble the deck". Reads each part file, works out which slide is which from its kicker and title, assigns streams, flags gaps, and writes an assembly report. Run gap-deck-review and gap-speaker-notes afterward. Output is a .pptx built on the I&G template plus a report.'
---

# GAP: Deck Assembly

Every other `gap-*` skill makes a few slides. This one puts them into the order the audience sees, builds the slides that belong to no analysis (cover, agenda, dividers, Next Steps, Questions), adds the standard I&G appendix, and tells the user what is still missing. It does not change the content of a part slide. Content checks (totals, stale facts, spelling) belong to `gap-deck-review`, which runs on the assembled deck.

Slides use `ig-deck-style` (required). Copy rules come from `improve-and-grow-brand`: no em dashes, colons between a name and its description.

## Workflow

1. **Collect the parts.** Ask only for what you cannot find: the folder with the slide files from the other gap skills, the client's full name, the revenue streams (in the order the deck should present them), and the presentation date. Ask who the presenter is; there is no default, because different team members build plans. Streams usually come from the Revenue Breakdown or Growth Goals files.
2. **Scan.** `python scripts/scan_parts.py FOLDER --streams "Stream A;Stream B" --json parts.json` prints each slide with the slot it fills and the stream it belongs to. Slides it cannot place inherit the previous slide's slot in the same file (marked "inherited"); read those lines. Fix anything wrong in `parts.json` by setting `slot`, `stream` or `slides` on the entry.
3. **Write `assembly.json`**: client, kicker, presenter, date, streams, parts (from the scan), and any static overrides. Schema and every option: `references/assembly-schema.md`.
4. **Assemble.** `python scripts/assemble_deck.py assembly.json Client_GAP_Deck.pptx --report assembly-report.md`. ERROR lines stop the build (a slide with no slot, or no stream in a multi-stream deck): fix them in the config. WARN lines are built anyway: missing slides, duplicates, appendix figures to verify, slides without notes.
5. **Look at it.** Render with `ig-deck-style/scripts/qa_render.py` and open the cover, agenda, dividers, Next Steps and appendix slides. Part slides were already checked by their own skills.
6. **Hand over.** Send the deck and `assembly-report.md`. In three or four sentences: slide count, what is missing (name the skill that builds each missing slide), the appendix figures the user must confirm, and the next step: run `gap-deck-review`, then `gap-speaker-notes` for slides without notes. The user imports the deck into Google Slides with Insert > Import slides.

## Standards

- **Order is fixed** (`references/blueprint.md`): cover, agenda, Section 1 Organization Overview and Growth Goals, Section 2 Market Research and Competitive Analysis, Section 3 Marketing Presence and Systems Audit, Section 4 Marketing Plan and Projected Results, Next Steps, Questions, appendix. Inside Section 2 each stream gets its own divider (skipped for a single stream), then Sales and Marketing Goals, optional brandscript, Keyword Research, Share of Voice, SEO Stats, Website Review. The Market Research Summary closes the section.
- **The agenda follows the sections that exist.** The agenda labels and the section dividers come from one list, so they always agree and always run in the same order. A section with no slides is dropped from both and the remaining sections are renumbered. Say so.
- **Missing is not an error.** A deck can go out without, say, a Local Search Audit. Report it as NOT INCLUDED with the skill that would build it. Never invent or stub a missing analysis slide.
- **Appendix figures go stale.** The I&G Overview stats, the camp logos and the Black Rock caption live in a static library (`assets/static-library.json`, built from approved I&G slides). The assembler lists each as VERIFY. Before sending, confirm every figure against current sources, and use `static_overrides` to change text without editing the library. When the user confirms new figures, update the library with `scripts/refresh_library.py` and its `last_verified` date.
- **The case study is a claim.** Do not assert a result you have not checked against the published case study. If the caption disagrees with the source, ask which is right.
- **Nothing is rebuilt.** Part slides are copied as they are, with their speaker notes. If one is wrong, fix it with the skill that made it and rerun the assembly.
- **Browser or accounts:** none needed. This skill only reads local files.

## Files

- `scripts/scan_parts.py`: identify slides and streams. `scripts/assemble_deck.py`: build the deck and report. `scripts/blueprint.py`: order and classification rules. `scripts/pptx_merge.py`: slide copy. `scripts/refresh_library.py`: replace appendix slides.
- `assets/ig-static-slides.pptx`, `assets/static-library.json`: I&G Overview, Our Approach, Camps We've Served, Black Rock story, and the optional Implementation Process.
- `references/blueprint.md`, `references/assembly-schema.md`
- `examples/make_sample_parts.py` (synthetic parts), `examples/sample-assembly.json`
- `evals/evals.json`

Requires python-pptx and the ig-deck-style skill. LibreOffice is only needed for rendering.
