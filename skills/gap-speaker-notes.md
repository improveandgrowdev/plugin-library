---
name: gap-speaker-notes
description: Adds and formats speaker notes for an Improve & Grow (I&G) Growth Action Plan (GAP) deck. Writes notes for content slides that have none (skips title, agenda, section break and closing slides), and restructures existing notes into an easy-to-scan format with bold labels, short paragraphs and bullet points (What it shows, How to read it, Talking points, Caveats, Sources). Checks that every figure and date in the notes is on the slide and that notes stay under the 4,000 character limit. Use whenever the user wants speaker notes, talking points, presenter notes, "notes on the slides", cleaned-up or reformatted notes, or a deck prepared for presenting, even if they only say "add notes" or "fix the notes". Run after the slides are built and before gap-deck-review.
---

# GAP speaker notes

Gives the presenter something to glance at mid-sentence. Two jobs: write notes for slides that need them and have none, and reformat notes that exist but are a wall of text. Notes are speech aids, so they are short, structured and traceable to the slide.

**Position in the workflow:** after the slide-making skills, before `gap-deck-review` (which checks that notes and slides agree).

## Inputs
- The deck as .pptx (attached, or exported from Google Slides with the Google Drive connector). Work on a copy; never overwrite the original.
- Whatever sits behind the slides, so notes can say more than the slide does: `model.results.json`, the keyword workbook, the audit exports, the plan config, the Client Overview. Look in the session and outputs first. If a slide's analysis is not available, write notes from the slide alone and keep to what it shows.

## Workflow
1. **Plan.** `python scripts/plan_notes.py deck.pptx notes/`. It prints one row per slide: class, notes status and the action, and writes `notes/context/slide-NN.txt` (all text on the slide plus any existing notes), `notes_plan.json` and `notes_existing.md`.
   - **skip:** title, agenda, section breaks, appendix dividers, closing slides. No notes.
   - **light:** static agency slides (Plan. Build. Grow., I&G overview, proof, roles). Short notes, talking points only, restating the slide.
   - **full:** everything else: data, findings, plan, budget, audits. Full format.
   - Status: `missing`, `wall of text`, `long paragraph`, `no structure`, `too long`, `ok`. Actions: write, reformat, keep, none.
   If the classifier is wrong for a slide (a section break with real content, say), decide by looking at it and say so in the reply.
2. **Look at the slides.** Render with ig-deck-style's `qa_render.py` and view the images for slides with charts, tables or screenshots. The text alone does not say what a bar chart shows.
3. **Write `notes.md`** with one `## <slide number>` block per slide that needs work.
   - Missing notes: write them from the slide and the analysis behind it. `references/notes-by-slide-type.md` says what to cover for each kind of GAP slide.
   - Existing notes: start from `notes_existing.md` (an automatic first split) and keep every fact, number and caveat. Restructure, tighten the wording, bold the lead-ins, turn lists into bullets. Cut only repetition and filler, and say in the reply if you cut anything of substance.
   - Format and length rules: `references/notes-format.md`. Before and after example: `examples/keyword-before-after.md`.
4. **Apply.** `python scripts/apply_notes.py deck.pptx notes/notes.md Deck_With_Notes.pptx`. It writes bold labels, bullets with a hanging indent and paragraph spacing into the notes of a copy, and only the slides listed. Add `--only-empty` to protect slides that already have notes.
5. **Check.** `python scripts/check_notes.py Deck_With_Notes.pptx --allow model.results.json`. Fix every Fix. For each "figure not on the slide", confirm it against its source or take it out; a number you cannot trace does not stay. Rerun until clean, or until only Check items you can justify remain.
6. **Deliver** the .pptx and a short summary: how many slides got new notes, how many were reformatted, which slides were skipped and why, and anything you could not source or that looks wrong on a slide (report it; do not fix slide content here). Then suggest running `gap-deck-review`.

## Rules
- Notes only. Never change slide content, layout or the original file.
- Every figure and date in a note is on the slide or comes from the analysis behind it. Do not invent, estimate or recombine numbers. Say "about" for rounded values.
- If the notes and the slide disagree, do not choose silently. Keep the slide's figure unless the model or source line shows it is stale, and flag the discrepancy.
- Voice: improve-and-grow-brand. Plain words, written to be said aloud, no em dashes, colons between a name and its description, findings framed as opportunities, no hype. Vary sentence length.
- Do not add notes to skip slides, and do not pad light slides.
- Google Slides: the text, labels and bullets carry over on import. Whether bold survives has not been verified, so the structure never depends on bold alone. Say this once when delivering for a Slides deck.

## Files
- `scripts/plan_notes.py`, `apply_notes.py`, `check_notes.py`, `notes_common.py`
- `references/notes-format.md`, `notes-by-slide-type.md`; `examples/keyword-before-after.md`; `evals/evals.json`

Requires python-pptx; LibreOffice and ig-deck-style for rendering.
