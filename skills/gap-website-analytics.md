---
name: "gap-website-analytics"
description: 'Analyzes a client''s Google Analytics 4 (GA4) data and builds the 3-slide Website Analytics summary for an Improve & Grow (I&G) Growth Action Plan deck: Performance Summary, Channel Breakdown, and Recommended Improvements. Also produces the written analysis: sessions, users, conversion volume and rate, top channels, engagement, seasonality, year-over-year change, top Organic Search and Direct landing pages, top Organic Social and Referral sources, and notes on Cross-network and Paid Search (including Google Ad Grant accounts). Use whenever the user wants a GA4 review, website analytics summary, traffic or channel analysis, conversion or key event review, seasonality or year-over-year view, or analytics slides for a GAP or proposal, even if they only say "look at their analytics". Any client. Reads GA4 in the browser by default (Claude in Chrome, the user signed in to Google Analytics), uses a GA4 connector if one is present, and falls back to uploaded GA4 CSVs. Output is a .pptx built with ig-deck-style.'
---

# GAP: Website Analytics

One year of website activity, read honestly, then shown in three slides. Slides use `ig-deck-style` (required, installed next to this skill). Voice and brand come from `improve-and-grow-brand` (`references/copy-voice.md`).

**Default: read GA4 in the browser** (Claude in Chrome, the user signed in to Google Analytics), as in the Google Ads audit. `references/capture-browser.md`. Second choice: a GA4 connector, only if its tools exist and the client's property is connected (`data-sources.md`, Route B). Backup: the user uploads GA4 CSV exports (Route C). All three produce the same CSVs, so everything after capture is identical. The skill pulls 24 or more months where it can, for seasonality and a prior-year comparison.

## What you produce

1. **A written analysis in chat** (the basis for the slides): session volume, conversion volume and rate, the top 5 channels, key engagement metrics, seasonality, year-over-year change, top landing pages for Organic Search and Direct, top sources for Organic Social and Referral, and anything notable on Cross-network and Paid Search. Structure and checks: `references/analysis-playbook.md`.
2. **Three slides**, kicker WEBSITE ANALYTICS on each, generic headlines so they fit any client:
   - **Performance Summary:** five KPI cards (sessions carry a year-over-year change when one is available), a sessions-by-channel bar chart (up to 10 channels), three takeaways.
   - **Channel Breakdown:** channels as rows, metrics as columns (top 5 plus an All channels row, with a Sessions YoY column when available), three insight cards.
   - **Recommended Improvements:** two groups of up to three bullets.

If the user asks only for slides, still do the analysis first. The slides are only as good as it.

## Workflow

1. **Capture.** Follow `references/data-sources.md`, which routes to the browser capture (`references/capture-browser.md`) by default.
   - Browser first. If the browser is not available, or the user names another route, check for a GA4 connector with `tool_search` for "Google Analytics", "GA4", "analytics", "run report", and find the client's property by name (confirm with the user only if more than one could match). With neither, ask for uploads.
   - Pull the full plan: the current 12 months of reports, the same channel report for the prior 12 months, GA4's total rows, and sessions by month for 24 months or more (by channel, with key events). Add daily data only to investigate spikes.
   - Save each result as a CSV with the GA4 column names. If a report fails on one route, try the next route for that report only and say which. Ask for uploads using the short list in `data-sources.md` when needed.
   - Date range: the last 12 full months unless the user says otherwise. State it back.
2. **Compute.** `python scripts/ga4_to_metrics.py <folder> --prior <prior folder> -o metrics.json --totals "..." --prior-totals "..." [--client-location "Town"]`. It recognises reports by their column headers. Read every WARN and NOTE: each names a missing report or a figure it could not trust.
3. **Analyze.** Follow `references/analysis-playbook.md`. These are not optional: the key event audit, the data quality checks, the seasonality read, and the year-over-year comparability check (key events often have a shorter history than sessions).
4. **Write the analysis in chat.** Concise, plain words, caveats stated directly.
5. **Draft `spec.json`** (`references/slide-spec.md`): the words for the three slides plus speaker notes. Numbers on slides come from `metrics.json`, never from memory. Every figure typed into copy is checked against it.
6. **Build.** `python scripts/build_analytics_slides.py metrics.json spec.json "<out>.pptx"`. Fix every ERROR and WARN, or tell the user which you left and why.
7. **Check by eye.** `python <ig-deck-style>/scripts/qa_render.py <out>.pptx <dir> --notes-required`, then view each slide image. Fix overflow by shortening copy before touching type size.
8. **Hand over.** Present the .pptx. In two or three sentences say what you pulled, what you flagged, and that the user imports slides with Insert > Import slides in the GAP deck.

## Rules that matter most

- **Audit the key events before quoting a conversion rate.** GA4's "key events" are whatever the property marks as key. Split them into real actions (registrations, forms, calls, bookings), downloads, engagement markers (five-minute timers, video plays, page views) and hiring. Report the as-reported figure and the real-action figure. The slide's last KPI is the real-action count, labelled in the client's words (`actions_label`).
- **Compare years only where the data allows.** Sessions and engagement compare cleanly. Key events and conversions do not if a key event was created partway through the prior period: `yoy.key_events_comparable` says so. Never present a year-over-year conversion change when it is false or unknown.
- **Seasonality comes from the monthly series,** not a guess. Give the peak, the typical month, the ratio, the month the ramp starts, and which channel swings most.
- **Client-facing slides lead with the opportunity,** positive and specific, not the problem. "Real actions are countable" beats "conversions are overstated". The chat analysis can be blunter.
- **Mark inference as inference.** Bots, staff traffic, and which channel suspect sessions sit in are inferred from patterns. Say "look like" or "likely", and say what would confirm it.
- **GA4's total can differ from the sum of the channel rows.** Pull the no-dimension totals and pass them with `--totals`. Say which you used.
- **Ad Grant accounts:** the Google Ads audit is separate work. Share what the analytics shows (volume, landing pages, how GA4 classified the traffic) and what to check in the account. Do not make claims about Grant compliance or spend.
- **Copy:** plain words, no em dashes, colons between a name and its description, no calendar years in slide copy (dates live in the source line), say inquiries, bookings or registrations, never "leads". See `improve-and-grow-brand/references/copy-voice.md`.
- Do not edit the ig-deck-style template or builder. The channel table is drawn in this skill's builder from native shapes.

## Files

- `scripts/ga4_to_metrics.py`: GA4 reports (CSV) to `metrics.json`, including seasonality and year over year.
- `scripts/build_analytics_slides.py`: `metrics.json` + `spec.json` to the .pptx, with validation.
- `references/capture-browser.md`: the browser capture procedure (default).
- `references/data-sources.md`: the routes, the pull plan, the connector route, the upload backup, column names.
- `references/analysis-playbook.md`: what to compute, the checks, how to interpret, how to frame.
- `references/slide-spec.md`: the spec fields and copy limits.
- `examples/`: the Pilgrim Pines metrics and spec, a worked example (built from uploads, so no year-over-year or seasonality block).
