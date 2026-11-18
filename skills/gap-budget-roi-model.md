---
name: gap-budget-roi-model
description: Build the 3-year marketing budget and ROI model for an Improve & Grow Growth Action Plan (GAP). Produces an Excel workbook with live formulas (Summary, Budget, Assumptions, one tab per revenue stream, optional lower-cost scenario tab), plus the Investment and 3-Year Results slides and speaker notes generated from the verified numbers. Use this whenever the user mentions a GAP budget, investment slide, ROI projection, 3-year results, program cost, setup vs ongoing fees, revenue streams, leads-to-customers math, keyword-based lead forecast, "what will this cost and what will we get back", a reduced-scope or "start with one stream" alternative, or asks to rebuild or update the numbers for a client like Pilgrim Pines, even if they never say "model" or "workbook".
---

# GAP budget and ROI model

Builds the budget-and-return model that sits behind every Growth Action Plan. The structure is the same for every client so any I&G Planner can open any model and find things in the same place. What changes per client is the data: services and tiers, revenue streams (names, pricing, sales process), keywords, rates and capacity.

A config file (JSON) holds all of that. `scripts/build_model.py` turns it into the workbook; `scripts/verify_model.py` recalculates and proves it is clean; `scripts/build_slides.py` makes the slides from the verified numbers so deck and workbook cannot drift apart.

One revenue stream is the common case; the model handles one to many. With a single stream the Summary, tabs and slides simply show one block, and segments, capacity and scenarios are optional. Start with the single-stream config (`examples/example-service-business.json` has the pattern, drop the second stream) and add streams only when the client truly sells separate offers with different keywords, pricing or launch timing.

## What is fixed vs flexible

Fixed (do not improvise, this is the consistency promise):
- Budget itemized by service, one-time setup separated from ongoing monthly, subtotals "of which setup / of which ongoing", credits, total invoiced, flat monthly billing, separately billed usage and paid media, total program cost.
- Lead math: addressable searches x seasonality, PPC and SEO each produce leads, leads become customers (skip the lead step for eCommerce-style streams), customers rebook, revenue = customers x value, ROI = revenue ÷ program cost.
- Three years, monthly engine, Years 1 / 2 / 3 and a 3-year total. Year 1 includes setup.
- Assumptions tab: every input blue on yellow with a source note; unconfirmed numbers marked PLACEHOLDER (red).

Flexible (per client):
- Which services, at which tier, with which fees. Omit what they will not buy; add a new website, custom integrations, a CRM, and so on. See `references/service-catalog.md`.
- Revenue streams: names, nouns, how sold, price, segments, rebooking, sales cycle, seasonality, capacity. See `references/stream-patterns.md`.
- PPC style (Google Ad Grant vs paid budget), launch timing per stream, scenarios.

## Workflow

1. **Gather inputs** (check what exists before asking: the client's GAP deck, Camp Growth Planner or client overview, Google Ads audit, keyword research exports, discovery call notes, I&G fee list, earlier models). Ask the user only for what you cannot find. You need:
   - Fees: services, tier, one-time setup, monthly, credits, any usage billed separately.
   - Each revenue stream: name, what is sold, average value, close rate, rebooking, sales cycle, capacity.
   - Keywords per stream and 12 months of search volume history for seasonality.
   - PPC: Grant or paid (budget, CPC); CTR by campaign if an audit exists; launch order of streams.
   - Start month and the Year 1 ROI target if any.
   See `references/assumptions-guide.md` for sources and defaults. When something is missing, use a moderate estimate and mark its note PLACEHOLDER rather than stalling.
2. **Seasonality**: `python scripts/seasonality.py history.csv --end-month 9` (or `--values`), once per stream.
3. **Write the config** by copying the closest example in `examples/` and editing. Schema: `references/config-schema.md`.
4. **Build**: `python scripts/build_model.py config.json Client_3-Year_Marketing_Budget_and_ROI.xlsx`
5. **Verify**: `python scripts/verify_model.py Client_....xlsx`. It must report 0 errors and no problems. It writes `Client_....results.json`. If a check says CHECK (capacity exceeded, Grant allowance exceeded), tell the user and adjust the assumption they would choose, do not silently change it.
6. **Sanity-read the results** before showing anyone. Does Year 1 ROI look right for the mix (setup drags it down)? Is any stream carrying most of the revenue (say so)? Is revenue per customer sensible? Spot-check one stream's first year by hand as `verify` does not judge plausibility.
7. **Slides**: `python scripts/build_slides.py results.json config.json out.pptx` makes the Investment slide, the 3-Year Results slide and one slide per scenario with speaker notes (ig-deck-style look). Render and look at them (`qa_render.py` from ig-deck-style). Copy into the GAP deck or send as is. Customize labels and notes through the optional config text fields.
8. **Deliver**: send the workbook (SendUserFile) and the slides. In the reply give: the 3-year revenue, cost and ROI line, the open PLACEHOLDERs that need client confirmation, and any risk the numbers carry (one stream dominating, capacity caps biting, Grant allowance close). Copy rules: improve-and-grow-brand voice, no em dashes, say "revenue" not "profit".

## Rules that keep the numbers honest
- Slides and write-ups take numbers only from `results.json`. Never retype figures from memory or from an older deck.
- ROI is gross revenue ÷ total program cost. Include separately billed usage and paid media in cost.
- Do not back-solve assumptions to reach a target ROI. If the target fails, show that and offer the levers (scope, timing, scenario).
- Round only for display. Counts on slides are whole numbers.
- State which inputs are the client's data, which are tool data and which are I&G placeholders.
- A change to one input should be a change to one cell. If you find yourself editing formulas, change the builder instead and re-run it.

## Reproduction test
`examples/pilgrim-pines.json` must reproduce revenue $426,281 / $1,032,426 / $1,269,232, program cost $64,100 / $56,100 / $56,100 and ROI 6.65X / 18.4X / 22.6X. Run it after any change to the scripts. `examples/example-service-business.json` (paid search, new website, custom integration, sales lag, credit, no LeadHub) covers the other path.

## Files
- `scripts/build_model.py`, `verify_model.py`, `build_slides.py`, `seasonality.py`
- `references/config-schema.md` (every config field), `model-structure.md` (tabs and logic), `service-catalog.md`, `stream-patterns.md`, `assumptions-guide.md`
- `examples/pilgrim-pines.json`, `examples/example-service-business.json`

Needs Python with openpyxl and python-pptx; recalculation uses the xlsx skill's `recalc.py` (LibreOffice). Slides need the ig-deck-style skill.
