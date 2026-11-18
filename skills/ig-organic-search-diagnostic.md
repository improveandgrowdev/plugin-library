---
name: ig-organic-search-diagnostic
description: Run a skeptical, evidence-first diagnosis of whether a client actually has an organic search performance problem, and if so what kind. Separates genuine ranking loss from search-demand shifts, SERP changes, CTR changes, keyword-mix changes, paid cannibalization, and conversion or tracking problems — then classifies the client (Healthy / Search environment changed / Recover / Rebuild / Not an SEO problem), cites the evidence, names one priority action, and names what not to waste time on. Use this for any client-level organic performance review, portfolio audit, "why are our rankings/traffic/leads down" question, churn-risk or retainer-justification analysis, QBR or monthly report prep, or when asked to compare year-over-year or period-over-period organic performance. Use it even when the user is confident the problem is SEO — testing that assumption is the point. Run ig-measurement-integrity-check first. Pulls all platform data via Windsor.ai.
---

# Organic Search Diagnostic

Diagnose before prescribing. A client's rankings can fall while their business is fine, and their rankings can hold while their business is failing. The job is to work out which situation you're in, and to be willing to conclude that there is no SEO problem.

**Run `ig-measurement-integrity-check` first.** It resolves which GSC property to trust, establishes the usable date window, and catches tracking breaks. Roughly half of all apparent performance declines turn out to be measurement artifacts, so diagnosing before that gate passes wastes the work. If it returned blockers, report those instead of a diagnosis.

## Data access

Pull Search Console, GA4, Google Ads, Google Business Profile, Bing, Meta, and LinkedIn data through **Windsor.ai** (`Windsor.ai:get_data`). Get field IDs from `Windsor.ai:get_options`; never guess them. Ahrefs is available for SERP-level and backlink questions GSC can't answer.

## Choose the comparison windows

The integrity check established what's available. Now choose deliberately.

Default to **6 months vs previous 6 months** as the primary lens, with a **YoY block** (usually 3–4 months, whatever fits the window) as a secondary check.

**Invert that for seasonal businesses.** Camps, retreats, tourism, lodging, event venues, landscaping, roofing, pools — for these, a sequential 6-month comparison spans peak season against off-season and is close to meaningless. Lead with the YoY block and say why. If the two lenses disagree in direction, that disagreement is itself the finding: report both and explain which one answers the business question.

Watch for a **second season**. A client can have a summer peak and a November–December peak (holiday events, gift cards). Check the monthly series for shape before assuming one cycle.

## Pull these dimensions

For each period: `clicks`, `impressions`, `ctr`, `position`, monthly. Then query-level (`query`), page-level (`page` — not `pagepath`, see below), `search_appearance`, and GA4 sessions plus conversions filtered to organic. Add Google Ads campaign-level clicks and spend whenever a paid account exists.

## Read the numbers correctly

Before interpreting anything, read `references/data-quirks.md`. It documents the traps that produce confident wrong answers — the October 2025 reporting discontinuity, why `position` improvements are usually mix shift rather than ranking gains, the unreliable brand/non-brand classifier, filter thresholds that fake disappearing keywords, `pagepath` collapsing distinct URLs, and what `search_appearance` can and can't tell you. Read it before writing conclusions, not after.

The single most important habit: **impressions are not demand and average position is not rankings.** Both are dominated by which queries a site happens to appear for, which changes constantly for reasons unrelated to performance. Clicks, CTR measured against stable impressions, and revenue are the trustworthy signals.

## Separate the causes

Work through these mechanically. The goal is to attribute the change, not to find the first plausible story.

**Search demand.** Did impressions on stable, specific queries fall? Brand-term impressions are the cleanest demand proxy — they're not affected by ranking. Brand impressions down with position unchanged means fewer people are searching for the client, which is an awareness problem, not a search problem.

**Ranking loss.** Look at position *per query*, not site-wide. Real ranking loss shows as position degradation on specific queries whose impressions held steady. If impressions moved too, you're probably looking at mix shift.

**CTR.** Position stable but CTR down means something is intercepting clicks above the result — AI Overviews, expanded local packs, the client's own ads, aggregators. GSC can't distinguish these; manual SERP inspection can.

**Keyword mix.** Did the site lose high-volume, near-zero-CTR head terms? That looks catastrophic in impressions and costs almost nothing in clicks. Compute clicks-per-lost-impression before treating it as damage.

**Paid cannibalization.** Mandatory whenever a paid account exists. Compare organic *and* paid clicks for the same query cluster across both periods, then sum them. Organic brand clicks falling while brand paid clicks rise means the client is buying back traffic they had for free — the combined figure is the real story, and it is often nearly flat when organic alone looks catastrophic.

**Conversion and tracking.** Traffic up with revenue down is a conversion problem. Conversions moving without sessions moving is usually a tracking change. The integrity check should have caught the obvious cases.

**Content and URL changes.** Compare page-level clicks. Look for URL taxonomy migrations (an old path pattern replaced by a new one), and for traffic *relocating* between pages rather than disappearing — a Google Business Profile link retargeted from the homepage to a subpage moves large volumes and looks like a homepage collapse.

## Test your own conclusion before writing it

This is the step that most changes output quality. Once you have a causal story, try to break it.

State the mechanism, then ask what else would have to be true. Then check that thing.

- "Duplicate UTM URLs split ranking signals and caused the decline." → Was the duplicate share actually different between periods? If it was 25.5% then and 25.0% now, it's a standing defect, not the cause.
- "Google canonicalized the tagged URL over the clean one." → Fetch the clean URL and read the canonical tag. If it's clean and self-referencing, the claim is wrong and those are local-pack clicks.
- "This keyword cluster disappeared." → Was there a click threshold on the query pull? Re-pull unfiltered before saying anything vanished.
- "Rankings improved." → Did impressions fall at the same time? Then average position rose because low-position impressions left the denominator.

Where a mechanism is visible but the cause isn't determinable from the data, say so and make the priority action a diagnostic step rather than a fix. Recommending tactics against an unidentified cause is how retainers get spent on the wrong thing.

Only attribute what you can support. If identified causes account for 60% of a decline, say 60% and name the remainder as unexplained. Inventing coverage is worse than admitting the gap.

## Classify

Read `references/classification.md` for the definitions, decision order, and worked examples from real clients. The categories:

- **Healthy** — no material change, or change explained by non-recurring events in the base period
- **Search environment changed** — real loss driven by Google-side changes outside the client's control
- **Recover** — genuine, addressable loss of visibility the client previously had
- **Rebuild** — visibility never existed or is too degraded to recover incrementally
- **Not an SEO problem** — the business problem is real but sits in conversion, tracking, demand, or paid

Classify against the business-relevant subset of traffic, not total clicks. A site whose blog traffic collapsed while its commercial pages grew is not declining in any way that matters.

## Output format

Per client:

```
## [Client] — [Classification]

[Seasonality/lens note if the windows disagree or the business is seasonal]

[Metrics table: clicks, impressions, CTR, sessions, revenue/leads — both periods and % change]

**Is there a real problem?** [Direct answer, with the numbers that settle it]

### Separating the causes
[Which mechanisms are operating and which are ruled out. Include the paid overlay
whenever paid exists. Note what remains unexplained.]

**Primary business problem:** [One sentence, in business terms not SEO terms]

### Evidence
1–3 numbered items, each a specific number or comparison

### One highest-priority action
[Single action. A diagnostic step if the cause isn't yet identified.]

### What not to spend time on
[Specific things, with the reason each is a dead end]
```

Include a portfolio-level section when running several clients — cross-client patterns are often the biggest finding, and a change appearing simultaneously across unrelated clients is a platform or reporting change, never a coincidence.

## Tone

Write for a skeptical analyst. Lead with the number that settles the question. Flag corrections to your own earlier claims explicitly rather than quietly revising. Don't hedge conclusions you can support, and don't assert ones you can't.

## Reference files

- `references/data-quirks.md` — GSC, GA4, and Windsor traps. Read before interpreting any numbers.
- `references/classification.md` — Category definitions, decision order, worked examples.
