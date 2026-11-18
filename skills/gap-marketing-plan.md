---
name: gap-marketing-plan
description: Builds the Marketing Plan section of an Improve & Grow (I&G) Growth Action Plan (GAP) deck. It documents the plan for the Build phase (website improvements, landing pages, ad campaign setup, SEO program setup, LeadHub or CRM setup, tracking and analytics setup) and the ongoing Grow phase for each revenue stream in focus, as I&G-style slides (strategy per stream, phased timeline for multi-stream plans, Build slide per stream, service Build slides, Grow stage) plus an optional written plan document. Use whenever the user wants the marketing plan, implementation plan, Build and Grow plan, phased launch plan, strategy slides, what we will build and when, or the plan section of a GAP, even if they only say "document the plan". Run it after gap-budget-roi-model, since the budget model already decides the streams, services, fees and launch months, so this skill starts from that config and builds out the detail of the work and the context around it.
---

# GAP marketing plan

Documents what Improve & Grow will do for the client and when: the Build phase (landing pages, ad campaigns, SEO setup, CRM, tracking, website work) and the monthly Grow phase, one stream at a time.

**Work order: budget first, plan second.** Run `gap-budget-roi-model` before this skill, even though the Marketing Plan appears before Investment in the deck. The budget model settles which services are provided, at what cost, which streams launch when, and what results to expect. This skill takes that as given and adds the detail and context: what gets built, in what order, and why the plan supports the budget and the projected results. The plan is never allowed to disagree with the model.

One revenue stream is the normal case. The skill handles one to four.

## Inputs
- **Required:** the budget model config (`budget.json`) and, once the workbook is calculated, `model.results.json` from `verify_model.py`. If the user has not built the budget yet, build it first with `gap-budget-roi-model`. Only continue without it if the user insists, and say that fees, results and months will need a cross-check later.
- Also useful: the GAP deck or Client Overview, the Growth Goals and Revenue Breakdown slides (goals), Google Ads and GA4 findings (tracking problems), the brand messaging guide status.

## What comes from the budget, and what is still asked
From the budget config, never asked again: client, start date, services in scope (derived from the fee lines and PPC mode), stream keys and names, ads-live month per stream (`launch_month`), fees and projected results. `scaffold_plan.py` fills these and sets default build windows (the month before ads go live, and the launch month).

Read the derived scope and the stream list back to the user in two short lines, for confirmation only: "Plan covers Youth Camps, Guest Groups and Campsites, with PPC on the Grant, SEO Premium, LeadHub with the Retreat Snapshot, analytics and sitewide website work. Right?" Two things can differ from the model and are worth confirming:
1. **Goals per stream.** Read back each goal in one line (from the Growth Goals and Revenue Breakdown slides). The Strategy slide shows one objective per goal.
2. **Streams that share one plan.** Budget streams that are one offer in the plan (for example Youth Camps and Family Camp) are merged with `covers`.

Then ask only what the budget cannot know, in one batch with a default for each (see `references/intake-questions.md`): which pages are built or updated per stream, campaign structure, the tracking events that count as real leads, website approach, client access, risks. Do not ask about fees, ROI, keywords or competitors.

## Workflow

1. **Confirm the budget exists** and note its path. Run `verify_model.py` if `model.results.json` is missing.
2. **Scaffold**: `python scripts/scaffold_plan.py budget.json plan.json` (add `--no-roadmap` for clients that are not camps or retreat centers). It drafts streams ordered by launch month, services, service Build slides and the Grow slide from `scripts/service_library.json`, and marks everything only the strategist knows as `PLACEHOLDER`.
3. **Fill in the detail**: replace every PLACEHOLDER with the real goals, pages, approach and takeaway cards, and merge streams with `covers` where the plan treats them as one. Adapt the library wording to the client's systems and tier (names of booking systems, counts of SEO projects). Schema: `references/config-schema.md`. Anything in the plan config overrides what the budget supplies, so only override months or scope when the budget model changes too.
4. **Check**: `python scripts/check_plan.py plan.json --budget-config budget.json`. Fix every warning, or explain it. A mismatch with the model means one of them is wrong: change the budget config and rebuild the model, never quietly diverge. Unresolved PLACEHOLDERs are warnings.
5. **Build the slides**: `python scripts/build_plan_slides.py plan.json Client_Marketing_Plan_Slides.pptx --budget-config budget.json`. Render with ig-deck-style's `qa_render.py` and look at every slide. Fix overflow (shorten items, drop a column) until the lint is clean.
6. **Optional written plan**: `python scripts/build_plan_doc.py plan.json plan.md --budget-config budget.json --results model.results.json`. The fees table and projected results come straight from the model, so the doc always agrees with the workbook. Read the generated month-by-month table and add detail where it is thin.
7. **Deliver**: send the .pptx (and the doc). In the reply, say how many slides, the stream order and ads-live months, and any warnings or open items (placeholders, client confirmations). Tell the user the slides import into Google Slides (Insert > Import slides). If the budget changes later, rerun steps 4 to 6.

## Standards that keep every plan consistent
- Fixed slide order: Roadmap (camps and retreat centers only), Strategy per stream, Phased timeline (only when there are 2 or more streams; a single-stream plan skips it), Build slide per stream, Build slides for services that need their own, Grow stage. See `references/plan-structure.md`.
- Each stream follows four steps: finalize the brand messaging guide, develop key landing page content, build the ad campaigns, configure conversion events. Campaign built in the first build month with no traffic; ads go live that month or the next on the first three pages; remaining pages follow.
- SEO, CRM, analytics and sitewide work start in Month 1 and support every stream. Conversion tracking is tested before any ad spend.
- Grow is monthly through Year 3: PPC audits and optimization, SEO audits with projects and content, monthly report and progress review, closed-loop tracking.
- Months and services must match the budget model. Same stream keys in both configs. The budget is the source of truth; the plan config only overrides it for detail.
- Copy rules: improve-and-grow-brand voice, no em dashes, colons between names and descriptions, no calendar years in headers, no invented numbers. Findings and plans are framed as opportunities.
- Every slide has speaker notes. The builder drafts them from the config; edit anything generic.

## Files
- `scripts/scaffold_plan.py` (draft plan from the budget), `build_plan_slides.py`, `build_plan_doc.py`, `check_plan.py`, `plan_common.py`, `service_library.json`
- `references/intake-questions.md`, `plan-structure.md`, `config-schema.md`, `service-library.md`
- `examples/pilgrim-pines-plan.json`, `examples/example-single-stream.json`

Requires python-pptx and the ig-deck-style skill (builder and `qa_render.py`).
