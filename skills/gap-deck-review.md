---
name: gap-deck-review
description: Final quality review of an Improve & Grow (I&G) Growth Action Plan (GAP) deck before it goes to the client. Finds inconsistencies between slides, stale information (old data pulls, past dates, numbers that no longer match the budget model), discrepancies, wrong totals, spelling and grammar errors, brand copy violations (em dashes), leftover template text, and formatting problems (overflow, overlaps, fonts, colors, missing notes), maps every finding to a slide number, and can apply approved text fixes to a copy of the deck. Use whenever the user wants the deck reviewed, proofread, checked, QA'd or audited, asks "is the deck ready", "check the numbers", "anything stale", or finishes a GAP, even if they only say "review the deck". Run it last, after all other gap skills.
---

# GAP deck review

The last step of the GAP workflow. The deck is assembled from slides built by different skills on different days, so numbers drift, dates age and names change. The other skills do not police that; this one does. It reads the whole deck against itself, against the budget model, and against the clock, then reports what to fix by slide number.

**Position in the workflow:** run after the other gap skills (organization overview, revenue breakdown, goals math, keyword research, competitor slides, budget model, marketing plan, audits, notes). Run it again after fixes until the Fix list is empty. Slide-making skills do not need their own stale-data logic; staleness is caught here.

## Inputs
- **The deck as .pptx.** If the user attached it, use it. If it is a Google Slides link, export it as .pptx with the Google Drive connector (`download_file_content`, export to pptx). Never edit the original; work on copies.
- **The model files when they exist** (look in the session and outputs before asking): `model.results.json` from gap-budget-roi-model (needed for the numbers-vs-model check), the budget config and the plan config.
- **Client name** (for name and title checks), **presented date** (the title slide usually says "presented on ..."; else today), and any names that must not appear (another client's name left from a template).

## Workflow
1. **Mechanical pass.** `python scripts/lint_deck.py deck.pptx review/ --client "Client Name" --results model.results.json [--as-of YYYY-MM-DD] [--not "Other Client,Competitor"] [--words "ClientCoinedWord"]`. It writes `review/findings.json`, `deck.txt` (slide-numbered text with notes), `deck.json` and `facts.tsv` (every $, X and % with context). It checks:
   - numbers: totals and growth in tables, one-digit or transposed near-duplicates, figures close to but not equal to the model's, model figures missing from the deck, speaker-note numbers that differ from the slide for the same item
   - stale info: Source-line dates older than 60 days, start months already past, spread between data pull dates
   - consistency: client-name misspellings, casing variants such as LeadHub and Leadhub
   - language: misspellings, repeated words, spacing, a/an, brand copy (em and en dashes, spaced hyphens, calendar years in headers)
   - leftovers and structure: TBD, placeholder or template text, hidden or duplicate slides, missing titles
   - formatting: shapes off the slide, probable overflow, overlapping boxes, off-brand fonts and colors, text under 9 pt, odd title sizes, missing notes
2. **Look at the deck.** Render it (`python <ig-deck-style>/scripts/qa_render.py deck.pptx review/render`) and view every slide image and the contact sheet. Read `deck.txt` in full. The lint cannot see a chart that collides with its labels or a recommendation that is out of date.
3. **Judgment passes.** Work through `references/review-checklist.md`: numbers (use `facts.tsv` and the pairs in `references/gap-deck-anatomy.md`), stale info, consistency, language and voice, visual, speaker notes, structure and appendix candidates. Add what you find to `manual.json`, and list any lint findings you checked and rejected under `dismiss`.
4. **Report.** `python scripts/build_report.py review/findings.json review/report.md --client "Client Name" --manual review/manual.json --rendered --sections-out review/sections.json`. Publish it as a Claude Doc: a lead sentence with the counts, a Fix table, a Check table, Notes, and what was checked. Use the docs tools and fill one section per update. Keep the doc to this; do not paste it into chat. Add `--rendered` only if you looked at the slide images.
5. **Fixes, only with approval.** Offer to apply the clear ones: spelling, repeated words, dashes, name variants, and numbers the model settles. Put them in `fixes.json` and run `python scripts/apply_fixes.py deck.pptx fixes.json Deck_Reviewed.pptx`. It edits a copy, keeps run formatting, and prints a change log with anything not found. Then rerun step 1 on the copy and confirm the Fix count dropped. Anything that needs a decision (which of two numbers is right, rewriting a slide, moving content to an appendix) stays in the report. For a Google Slides deck, the report is the fix list; the reviewed .pptx can be re-imported, but say plainly that edits made in Slides since the export will not be in it.
6. **Reply** in a few lines: counts by severity, the three things that matter most, the doc link, and whether the reviewed copy is attached. State which passes were not done (for example no model results supplied, so numbers were not checked against the model).

## Rules that keep the review trustworthy
- Report only what you found. Do not say the deck is clean for a pass you did not run. A lint with no findings is not a review; the render and the read-through are.
- A finding needs a slide number (or "Deck"), the evidence, and a suggested fix. When two figures disagree, check the model, the source line and the notes before saying which is wrong. If nothing settles it, list both and ask.
- Fix means clearly wrong. Do not inflate severity; a long list of Notes buries the real problems.
- Dismiss false positives openly. The lint is heuristic: overflow and overlap estimates and near-duplicate numbers can be wrong, especially for scenario slides that legitimately show nearby figures.
- Apply no fix the user has not approved, and never overwrite the original file.
- Copy rules apply to anything you write into the deck: improve-and-grow-brand voice, no em dashes, colons between names and descriptions.

## Files
- `scripts/lint_deck.py` (with `extract_deck.py`), `build_report.py`, `apply_fixes.py`
- `references/review-checklist.md`, `gap-deck-anatomy.md`, `dictionary.txt` (words the spell check accepts; add client and industry terms)
- `evals/evals.json`

Requires python-pptx; pyspellchecker for the word-level spell check (`pip install pyspellchecker`); LibreOffice and the ig-deck-style skill for rendering.
