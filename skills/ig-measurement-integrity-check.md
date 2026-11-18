---
name: ig-measurement-integrity-check
description: Verify that a client's Google Search Console and GA4 data can actually support a performance conclusion, before anyone draws one. Checks the available data window, resolves which GSC property to trust, and hunts for dead conversion events, events that appeared mid-period, non-standard UTM tagging that breaks channel reporting, hacked-site spam in the query data, and paid campaigns absorbing organic demand. Use this whenever you are about to analyze, report on, or draw conclusions from organic search or GA4 data for a client — including monthly reporting, QBR prep, retainer justification, churn-risk review, or any "why is traffic/leads down" question. Also use it on its own when someone asks whether a client's tracking is working, whether conversions are being recorded, or why GA4 and Search Console disagree. Run this BEFORE the ig-organic-search-diagnostic skill, which depends on its output. Pulls all platform data via Windsor.ai.
---

# Measurement Integrity Check

Most apparent SEO problems are measurement problems. Before anyone concludes that a client's organic performance is declining, verify that the data is capable of showing what it appears to show.

This skill produces a short verdict: **the data can support a diagnosis**, or **these specific things must be fixed first**. It does not diagnose performance. That's `ig-organic-search-diagnostic`, which should not run until this passes or the user explicitly waives a blocker.

## Data access

Pull Google Search Console, GA4, Google Ads, Google Business Profile, Bing Ads, Meta Ads, and LinkedIn Ads data through **Windsor.ai** (`Windsor.ai:get_data`). Call `Windsor.ai:get_connectors` first to list accounts, and `Windsor.ai:get_options` for a connector's field IDs — field names must come from there, never from memory.

Two mechanics that will otherwise cost you time:

- **`get_data` responses can exceed the context limit.** When that happens the result is written to a file and the path is returned. The file is *double-wrapped*: a JSON array containing one object with a `text` key, whose value is a JSON **string** of the actual rows. Unwrap it before aggregating:
  ```python
  outer = json.load(open(path)); rows = json.loads(outer[0]['text'])
  ```
- **Grouping by a query-derived field silently drops rows.** GSC anonymizes rare queries, so query-level pulls typically cover only 30–40% of total clicks. Never reconcile query-level sums against site totals and conclude something is missing.

## Step 1: Establish the usable data window

Google Search Console retains roughly 16 months. In practice the Windsor connector returns about 15. This moves forward every month, so compute it — never hardcode a window.

Probe for the real boundary: request a wide range, and on `date_from cannot be more than 16 months in the past`, step `date_from` forward a month until it succeeds.

State the consequences plainly, because they constrain every comparison downstream:

- **True 12-month-vs-12-month year-over-year is impossible for any client**, from any tool. This is a Google retention limit, not a Windsor one, so Ahrefs' GSC integration can't get around it either. The only route to real annual YoY is a warehouse that's been storing GSC data independently (check Databox).
- **6 months vs previous 6 months** fits comfortably.
- **YoY** works only on the trailing block that fits inside the window — typically 3–4 months.

If a client's GSC property returns no data before a certain month while GA4 shows organic sessions in that period, the property was verified late or the domain changed. Flag it: that client has no YoY at all, and the gap is itself a finding.

## Step 2: Resolve the GSC property

Portfolios accumulate duplicate properties for one site — `sc-domain:example.com`, `https://example.com/`, `https://www.example.com/`, `http://example.com/`, plus per-subpath properties. **They return different numbers for the same site.** Picking wrong manufactures a decline.

Pull a monthly clicks series for every candidate, discard the empty ones, and name the property you chose in the output. Where a site is split into subpath properties, a root URL-prefix property covers all of them — prefer it, and say so.

## Step 3: Audit conversion events

This is where the highest-value findings live. Get the property's conversion field list from `Windsor.ai:get_options` on the `googleanalytics4` connector — event-specific fields appear as `conversions_<event_name>`. Pull them monthly alongside `sessions`, filtered to organic:

```
filters: [["session_default_channel_group", "eq", "Organic Search"]]
```

Look for four distinct failure modes:

**Dead events.** An event reading *exactly* zero for two or more consecutive months while sessions continue is a tracking break, not a demand change. Report the month it stopped. Zero is a much stronger signal than a large decline — real demand declines are noisy, broken tags are clean.

**Events that appeared mid-period.** A new event inflates total `conversions` and creates a fake improvement. If total conversions rose while individual events were flat, check whether a new `conversions_*` field starts partway through the window. Never report an aggregate conversion increase without confirming the event set is unchanged across both periods.

**Metric definitions that changed.** Watch for `transactions` moving sharply while `totalrevenue` holds, or average order value jumping. That combination means the purchase event was reconfigured, not that the business changed. Compute revenue per transaction for both periods and compare.

**Events that were never real.** Two patterns: an event that has never fired across the whole window (the client cannot measure that outcome at all — check whether they're spending ad budget against it), and an event that fires but doesn't represent the outcome it's named for. Landing on a contact page is not a lead. Say so, because it means the historical baseline is inflated too and fixing tracking will look like a decline.

## Step 4: Check channel classification and UTM tagging

Non-standard UTM parameters on links the client controls will fragment their own reporting.

Pull GSC pages filtered to tagged URLs:

```
fields: ["page", "clicks", "impressions"]
filters: [["page", "contains", "utm_"]]
```

Then pull GA4 `session_source_medium` by month to see how those land.

What to look for:

- **A non-standard `utm_medium`.** Anything other than `organic` creates a new source/medium row. `google / organic` will then diverge sharply from the Organic Search channel group — one client showed −47% on source/medium against −23% on the channel group. Anyone reporting off source/medium has been showing the client wrong numbers.
- **Multiple conventions live at once.** Google Business Profile has several linkable fields (Website, Appointment, Menu, Services, links inside Posts), so tagged URLs pointing at different pages with different campaign names usually means several profile fields were edited at different times by different people. Inventory them all; fixing one re-fragments the data.
- **A year baked into the campaign name** (`gbp-2026`). Guarantees the data breaks every January.
- **`utm_id`** used as a label. In GA4 it populates Campaign ID and is meant for paid reconciliation; it can collide with Google Ads data import.
- **Malformed strings.** A source/medium value containing raw `&utm_medium=` means a link's query string was mangled. Low volume, but it confirms the links are hand-built.

Before calling tagged URLs a duplicate-content problem, **fetch the clean URL and read its canonical tag.** If the canonical is clean and self-referencing, there is no indexing problem — the tagged clicks are almost certainly Google Business Profile "Website" button clicks from the local pack, which GSC attributes to the exact URL in the profile, query string included. That looks like a tagged URL outranking a clean one and isn't. Confirm before asserting.

The recommended convention, which keeps GBP clicks inside Organic Search so channel-level comparisons survive:

```
?utm_source=google&utm_medium=organic&utm_campaign=gbp
```

Add `&utm_content=<location-slug>` for multi-location clients. Drop `utm_id`. Keep years out of campaign names.

## Step 5: Check for spam and paid overlap

**Spam.** Scan the query data for terms unrelated to the client's business, especially non-English gambling, pharma, or streaming terms. A hacked site inflates the historical impression baseline and makes cleanup look like a decline. Trace the affected months to date the incident and confirm remediation rather than assuming it.

**Paid overlap.** Check `Windsor.ai:get_connectors` for a `google_ads` or `bing` account matching the client. If one exists, pull campaign-level clicks, impressions, and spend for both comparison periods. A brand campaign scaling up will absorb organic brand clicks — organic looks like it collapsed while total brand capture barely moved. Any organic conclusion drawn without this overlay may be wrong in direction, not just magnitude. Note whether the overlay is needed and whether the accounts exist.

## Output

Keep it short. This is a gate, not a report.

```
## Measurement integrity: [client]

**Property:** [chosen GSC property, and what was rejected]
**Usable window:** [start]–[end]. YoY available: [block or "none"]
**Paid overlay:** [required / not applicable] — [accounts]

| Check | Status | Detail |
|---|---|---|
| Data window | | |
| Property resolution | | |
| Conversion events | | |
| Channel/UTM integrity | | |
| Spam signature | | |

**Blockers** — fix before any performance conclusion:
1. [what, with the number that proves it]

**Cautions** — proceed, but constrain the claims:
1. [what, and which claims it invalidates]

**Verdict:** [Clear to diagnose / Diagnose with stated constraints / Not diagnosable until blockers fixed]
```

A blocker means a performance conclusion would be unsound — dead events on the client's primary conversion, a property that can't be resolved, or a channel definition that changed mid-window. A caution constrains specific claims without stopping the work.

If the verdict is "not diagnosable," say what to fix and stop. Don't soften it into a diagnosis with caveats — that's how a measurement failure gets reported to a client as a performance failure.

## Worked signals

These are real patterns worth recognizing:

- Every conversion event reads exactly 0 from one month onward, for eight straight months, while sessions continue. → Tracking broke that month, probably a site relaunch. No performance judgment is possible. The one event that had been firing was "landed on contact page," so the prior baseline was inflated too.
- Total conversions +22%, but `conversions_phone_call` starts from zero mid-window and transactions halve while revenue stays flat. → Two separate measurement changes; both the increase and the decrease are artifacts.
- An event named for the client's highest-value outcome (retreat inquiry, quote request) reads zero across the entire window while a paid campaign spends four figures a month driving that exact outcome. → The client cannot measure the thing they're buying.
