---
name: gap-revenue-breakdown
description: 'Builds the Revenue Breakdown slide for an Improve & Grow (I&G) Growth Action Plan deck: a table of the client''s current annual revenue by major revenue stream, the unit volume for the same period, the future annual revenue goal, and the year-over-year change, with a totals row. Use whenever the user wants the revenue breakdown, revenue by stream or service, revenue goals table, or the Camp Growth Planner slide for a GAP or proposal presentation. Works for any client: stream names, the number of streams, and the unit volume metric (guests, events, jobs, students) are the client''s own. Output is a .pptx slide built with ig-deck-style, ready to import into the I&G Google Slides deck.'
---

# GAP: Revenue Breakdown

One table that shows where the client's money comes from today and where the goals are headed. Slides use `ig-deck-style` (required); voice rules come from `improve-and-grow-brand`.

## What you produce

**REVENUE BREAKDOWN** slide, title "Annual Revenue by Stream" unless the user prefers another (for example "Camp Growth Planner"). A native, editable table:

| Revenue Stream | Current Annual Volume | Current Annual Revenue | Future Annual Revenue Goal | YOY Change |
|---|---|---|---|---|

One row per major revenue stream, then a Totals row (revenue, goal and total YOY change; the volume cell stays blank because the units differ by stream). YOY change is shown with a sign, for example +30%.

The headers carry no calendar year, so the slide stays valid whatever year it is built. The stream names, how many streams there are, and the unit volume metric are the client's own. Camps call the same stream by different names (Camp Programs, Youth Camp, Summer Camp), so use the client's language and group minor streams so the table stays at 10 rows or fewer. Details and the JSON spec: `references/slide-spec.md`. Where the numbers come from: `references/source-map.md`.

## Workflow

1. **Gather.**
   - Camps, retreat centers and conference centers: the Camp Growth Planner report. The summary PDF collapses each stream, so you need the expanded sections or the planner link for volumes and revenue. Also check ClickUp and the user's notes for newer client-confirmed actuals.
   - Other clients: revenue by service or product line for the last full year or trailing 12 months (a P&L, QuickBooks summary or client-provided list) and the matching unit volume (jobs, orders, students, projects). Ask the user if these are not in the Client Overview or the files.
   - Goals: the annual revenue goals agreed with the client or set in the planner. If a goal is missing for a stream, ask rather than assume.
2. **Draft the spec** (`references/slide-spec.md`). One row per stream with `stream`, `volume` (the number and unit as text, for example "113 events, 3,698 guests"), `revenue`, `goal`. Name the sources.
3. **Run the builder:** `python scripts/build_revenue_breakdown.py spec.json "<out>.pptx"`. Resolve every WARN: fix the spec from the sources, or tell the user both numbers and which source says what. Never pick one silently.
4. **Check by eye.** Render with `ig-deck-style/scripts/qa_render.py` and view the slide.
5. **Hand over.** Present the .pptx and say in two or three sentences which sources you used, any stream grouping you chose, and anything you flagged. The user imports the slide with Insert > Import slides.

## Rules

- **Use the newest actuals.** If a revenue figure changed after a report was generated (for example the client sent final campsite actuals), use the new figure and recompute the totals and the total YOY change. Say you did. A stale total is the most common error here, and the builder warns when the rows do not add to it.
- **Same period for revenue and volume.** Record it in `period` (for example "2026", "FY2026" or "Last 12 months") and the goal year in `goal_year`. They go into the speaker notes only, not the headers.
- **Volume is plain text with its unit,** as the client would say it. Streams without a meaningful unit use "n/a".
- **YOY change is computed** from goal and current revenue. If you supply a value that disagrees, the builder warns.
- **Totals are sums.** Do not take the total from a report without checking it against the rows.
- This slide has no capacity or utilization columns. Capacity is a separate analysis.
- Plain words, no em dashes (see `improve-and-grow-brand/references/copy-voice.md`).

## Handoff

Keep the spec file. The goals-math skill (`gap-growth-goals-math`) uses the same current revenue and goal figures, so the two slides never disagree.
