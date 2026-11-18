---
name: "gap-website-audit"
description: "Runs a rigorous, section-by-section website audit for any business using Improve & Grow's Website & Content Audit framework, then delivers an audit score workbook and a two-slide GAP summary."
---

# GAP: Website Audit

Audit a client's website against the I&G framework, with the user checking each section, then produce two files: the audit score file (a designed workbook with everything in it) and a two-slide summary that tells the client what is working and where to start. Slides use `ig-deck-style` (installed next to this skill). Voice comes from `improve-and-grow-brand/references/copy-voice.md`.

The framework is `references/framework.md` (generated from the workbook by `scripts/extract_framework.py`). It controls section order, audit points, point descriptions and weights. Every audit has the same fixed sections plus one **Revenue Stream: [name]** section for each revenue stream the client has, all scored on the same six points. Do not invent criteria, merge points, reorder or skip sections. If the user uploads a different or updated audit workbook, run `extract_framework.py` on it and use that output instead.

Read `references/scoring-rules.md` before rating anything. Read `references/evidence-guide.md` when you start gathering evidence. `references/audit-json.md` describes the record you keep and the outputs.

## Workflow

1. **Kick off, in one message.** Confirm only what you cannot find yourself: the site URL, the business or growth goal that should shape the lens (the GAP revenue goal if there is one), and these two questions:
   - **Revenue streams.** What are the core revenue streams to audit the content for? Every business has at least one: a camp might have Summer Camp Programs, Overnight Guest Groups and Campsite Rentals; a remodeler might have Whole-Home Remodeling and Kitchen and Bath. Each stream gets its own section, named "Revenue Stream: [name]", scored on the same points. If the user is unsure, propose streams from the site navigation and the GAP revenue breakdown and ask them to confirm.
   - **Proof of work.** What form does "showing the work" take for this client? Offer these four examples if the user is not sure: a facilities section (camp), a project portfolio (home remodeling contractor), a project gallery (interior designer), case studies (consultant). Record the answer in `proof_of_work_form`. It tells you what to look for when you rate the Proof of Work point.
   Also ask whether the business is location-based. Say what you cannot see from here (visuals, mobile layout, live device behavior) and ask for desktop and mobile screenshots of the home page and two or three inner pages, plus PageSpeed, accessScan and Safe Browsing results if they have them. Offer defaults so the answer can be one line. Do not ask anything you can look up.
2. **Start the record.** Create `audit.json` (see `references/audit-json.md`). Build the page inventory first (`references/evidence-guide.md`). Put the stream names in `revenue_streams`. Mark whole sections that do not apply in `sections_na` with a reason, and tell the user which you excluded and why.
3. **Work section by section, in framework order:** User Experience; Header, Footer, Home Page, About, then one Revenue Stream section per stream (in the order the user listed them), then Other Content (which includes Proof of Work); Technical SEO; Conversion Optimization. For each:
   - Gather the evidence that section needs (for User Experience, run the PageSpeed step below). Review the full page set before rating a multi-page section.
   - Show what you observed, briefly, only when it helps the user confirm you looked at the same thing. Observed content has no evaluation in it.
   - Give a table of every point in the section: point, rating (Strong, Moderate or Weak), and a one to three sentence observation. Then the section score. Run `python scripts/check_audit.py audit.json --partial` for the number.
   - Ask for corrections in one line. Move on if there are none. Treat factual corrections as high priority: state the old rating, the new one and why, and log it in `revisions`.
   - Save `audit.json`. Do not show the `action` (recommended next step) field in chat. Add it for every non-Strong point when you save, so the chat stays short.
   - If the user says to run through the rest without stopping, do so and let them review everything at the end.
4. **Close the audit, without being asked.** Fill the closing fields (`references/scoring-rules.md`), then show in chat: 5 positive observations (4 to 5 words each), 5 critical observations (4 to 5 words each), and a key takeaway that fits on ONE line of the slide (75 characters or fewer).
5. **Build the files.** Run `scripts/check_audit.py` and fix every error and warning, or tell the user which you left and why. Then:
   - **Summary tab order fix (apply before building the score file).** In `scripts/build_score_file.py`, the "Score by area" table on the Summary tab must list the Content sections (Header Area through Other Content) directly under Content and before Technical SEO. If the script still builds `areas` as the four categories followed by all the Content sections, replace those lines so the loop is: for each category, append the category row, and if the category id is `content`, append its sections right after it. Apply this to the working copy of the script. The Scorecard tab is already in the right order.
   - `python scripts/build_score_file.py audit.json "<client>-website-audit-scores.xlsx"` (the score file: audit criteria, rating dropdown, evidence observed and recommended improvements for every point). Open the Summary tab and confirm the order is User Experience, Content, then the Content sections, then Technical SEO, then Conversion Optimization.
   - `python scripts/build_audit_deck.py audit.json "<client>-website-audit.pptx"` (two slides: Overview, Where to Start)
   - `python <ig-deck-style>/scripts/qa_render.py "<client>-website-audit.pptx" <dir> --notes-required`, then view both slide images. Fix overflow by shortening copy before touching type size.
6. **Hand over.** Present both files. In a few sentences: the overall score, what the slides lead with, anything flagged (provisional AI Discoverability criteria, unverified points, sections excluded), and that the slides import into the GAP deck with Insert > Import slides. The score file holds every rating, observation, next step and PageSpeed result, with live scores and dropdown ratings. If the user edits it, run `python scripts/sync_from_score_file.py <score file> audit.json` (it prints what changed), then `check_audit.py`, then rebuild both files.

## PageSpeed (User Experience section)

Do not ask the user for PageSpeed scores. Use the `pagespeed-insights-report` skill:

1. Find the five most visited pages (Ahrefs top pages or GA4 if connected; otherwise the home page and the four pages the main navigation points to most prominently, and say that is what you did).
2. Read `pagespeed-insights-report/SKILL.md` and follow it for each URL, one at a time. It returns mobile and desktop scores for Performance, Accessibility, Best Practices and SEO, plus Core Web Vitals and fixes.
3. Record the scores, report links, Core Web Vitals and the top fixes in `pagespeed` and `pagespeed_fixes` in audit.json (five desktop rows, five mobile rows). Use those results, and the Core Web Vitals, to rate Desktop Performance and Mobile Responsiveness (`references/scoring-rules.md`).
4. If a result has a Lighthouse warning, or a value could not be read, say so and record "n/a" rather than a guess. If the built-in browser is unavailable, say so and only then ask the user to paste results from pagespeed.web.dev.

## Rules that matter most

- **Rigorous, not generous.** A Strong rating has to be earned. Presence is not optimization. Details and the caps are in `references/scoring-rules.md`.
- **Never score what you did not verify.** No assumption-based scoring. If you could not see it, ask, or set the point to `unverified`. Never describe an element as present on typical-site logic.
- **Use each tool only for its own points.** PageSpeed does not judge readability or design.
- **No speculative words in observations** (likely, probably, implied, appears to suggest, seems intended to).
- **Never invent a number, a quote or a fact.** Every figure traces to a tool result or something the user told you.
- **Same standard for every stream.** Do not tailor or branch the six revenue stream points by business type. Read the wording inclusively: "offerings" means programs, services, packages or projects. Name the client's own terms in observations.
- **Provisional criteria.** AI Discoverability has no description in the template. Ask the user to confirm or replace the criteria the first time you reach it. Until then the deck says its criteria are provisional.
- **Frame the slides as opportunities.** They are read by the client. Findings and next steps are constructive. The score file observation can be blunter.
- **Copy:** no em or en dashes, no calendar years in slide copy, no "leads", plain words. The check script enforces these.
- **Concise with the user.** One question at a time, no over-explaining, caveats stated plainly.
- Do not edit the ig-deck-style template or builder.

## Files

- `scripts/extract_framework.py`: workbook to `references/framework.json` and `framework.md`.
- `scripts/audit_lib.py`: loading, validation and scoring shared by the scripts.
- `scripts/check_audit.py`: validate and print scores. `--partial` for mid-audit.
- `scripts/build_score_file.py`: audit.json into the score file (built from the framework, not copied from the template).
- `scripts/sync_from_score_file.py`: reads the user's edits in the score file back into audit.json.
- `scripts/build_audit_deck.py`: audit.json into the two-slide .pptx.
- `assets/camp-retreat-website-audit-template.xlsx`: the I&G template the fixed sections were extracted from (its Camp, Retreats and Facilities sections are replaced by the revenue stream template and Proof of Work). Only `extract_framework.py` reads it.
- `references/framework.md`, `framework.json`: the audit points.
- `references/scoring-rules.md`, `evidence-guide.md`, `audit-json.md`, `template-notes.md`.
- `examples/example-audit.json`: a complete fictional audit showing the shape.