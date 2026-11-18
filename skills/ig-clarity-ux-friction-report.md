---
name: ig-clarity-ux-friction-report
description: Use this skill whenever the user wants a 30-day UX friction summary from Microsoft Clarity — rage clicks, dead clicks, excessive scrolling, quick backs, and JavaScript errors. Trigger for requests like "run the Clarity friction report," "check for UX issues in Clarity," "how many sessions had rage clicks last month," or "give me a Clarity health check" — even if the user doesn't name all five metrics explicitly, as long as they're asking about Clarity session-level friction/UX signals over a recent period. Produces a markdown table (Name, Sessions Affected, % of Sessions Affected, Recommendations) built from Clarity's analytics dashboard data only. Does NOT pull session recordings or heatmaps as part of the default report — those are only used when the user explicitly asks to dig deeper into a specific flagged issue.
---

# Clarity UX Friction Report

Produces a 30-day summary of five Microsoft Clarity behavioral-friction signals — Rage Clicks, Dead Clicks, Excessive Scrolling, Quick Backs, and JavaScript Errors — as a single markdown table, sourced entirely from Clarity's analytics dashboard data (never recordings or heatmaps, unless the user asks to dig deeper afterward).

## Required tools

Call `tool_search` for `"clarity"` first if the Microsoft Clarity MCP tools aren't already loaded. This connector exposes exactly two tools:

- `query-analytics-dashboard` — natural-language query against the project's analytics/metrics dashboard. **This is the only tool this skill uses.**
- `query-documentation-resources` — Clarity docs lookup. Only use if you genuinely need to confirm a metric definition; not part of the normal flow.

There is no session-recordings or heatmap tool on this connector. If the user asks to "dig deeper" into a flagged issue, see "Digging deeper" below — it uses `query-analytics-dashboard` with narrower natural-language queries (e.g. broken down by page/browser/device), not a separate recordings tool.

If `tool_search` doesn't surface a Clarity server at all, tell the user their Clarity MCP connector isn't visible in this session and ask them to check/reconnect it, rather than guessing at data.

**One connector = one site.** Each Clarity MCP connection is tied to a single Clarity project's API token — it is not shared across every site in the account. If the user manages multiple sites in Clarity, running this report for "Site B" while only "Site A"'s connector is active will silently return zero/near-zero results rather than an error. See step 1 of the workflow below for the sanity check to run before trusting the numbers.

## Critical lesson: ask for session counts explicitly, per metric

`query-analytics-dashboard` is a natural-language wrapper, and how you phrase the query changes what it returns:

- A combined/aggregate-style query (e.g. "rage clicks, dead clicks, quick backs... for the last 30 days") returns **total event counts**, not sessions affected. These can be wildly different from the session-level number — e.g. one test run returned "36 total rage clicks" as an aggregate, but the real distinct-session count was only 7. Do not use aggregate totals for the "Sessions Affected" column.
- The correct phrasing is **one query per metric**, explicitly asking for distinct/unique sessions, e.g. *"Number of distinct sessions that had at least one rage click in the last 30 days."* This reliably returns a field like `DistinctSessionsWithRageClicks`.

This also means the 30-day range does **not** need to be chunked into smaller windows — the wrapper handles a full 30-day natural-language range natively in a single call per metric. A full report costs 6 calls total (5 metrics + 1 total-sessions query), which is well within any daily quota. Don't chunk by date and don't combine metrics into one query.

## Workflow

### 1. Confirm which site's connector is active — this is 1:1, not account-wide

**Important:** a Microsoft Clarity MCP connection is scoped to a single Clarity project via that project's API token. It is not account-wide — connecting once does **not** give access to every site in the Clarity account. Querying a domain that isn't the one behind the active token won't error; it will silently return zero sessions, which looks like "no data" rather than "wrong project."

Before running the report:
- Ask the user which site they want, if it's not already clear.
- Confirm (or verify, if there's any doubt) that the **currently connected Clarity connector actually corresponds to that site** — e.g. by running a quick sanity check like "show a sample of recent page URLs from the last 30 days" and checking the domain in the results — before pulling the full report.
- If the user wants a different site than the one currently connected, tell them plainly: they (or whoever manages the connectors) need to set up a **separate connector instance** for that site, using that site's own Clarity API token (from that project's Settings → Data Export in Clarity). Distinctly named connectors per site (e.g. `clarity-[Client]`, `clarity-ceilingoutfitters`) avoid collisions. Don't attempt to pass a "switch project" parameter — the tools don't support one; the token itself determines the project.
- If a report is requested for a site with no connector set up yet, say so clearly and stop rather than running the report against whatever project happens to be connected.

### 2. Get total sessions in range

Call `query-analytics-dashboard` with a query like:

> "Total number of sessions in the last 30 days."

This returns a field like `DistinctSessionCount` — this is the denominator for every percentage below.

### 3. Query each metric individually

Make **five separate calls**, one per metric, each phrased as "distinct sessions that had at least one X in the last 30 days":

- "Number of distinct sessions that had at least one rage click in the last 30 days."
- "Number of distinct sessions that had at least one dead click in the last 30 days."
- "Number of distinct sessions that had at least one instance of excessive scrolling in the last 30 days."
- "Number of distinct sessions that had at least one quick back in the last 30 days."
- "Number of distinct sessions that had at least one JavaScript error in the last 30 days."

Each returns a single count field (naming varies slightly, e.g. `DistinctSessionsWithRageClicks`, `DistinctSessionsWithDeadClicks`, `DistinctSessionsWithExcessiveScroll`, `DistinctSessionsWithQuickBack`, `DistinctSessionsWithErrors`) — read the value regardless of exact field name.

If any single query errors out, retry once with slightly different phrasing before giving up on that row.

### 4. Compute percentages

For each metric: **% of sessions affected** = (sessions affected ÷ total sessions from step 2) × 100, rounded to 1 decimal place.

### 5. Build the table

Output **only** a markdown table with these four columns, one row per metric, plus a one-line note above it with the date range and total session count:

| Name | Sessions Affected | % of Sessions Affected | Recommendations |
|---|---|---|---|
| Rage Clicks | 7 | 0.1% | Very low volume — worth a quick look at which page/element, but not a priority. |
| Dead Clicks | 396 | 5.6% | Check for unresponsive or slow-loading interactive elements on the affected pages; users are clicking and getting no feedback. |
| Excessive Scrolling | 2 | 0.0% | Negligible — no action needed right now. |
| Quick Backs | 628 | 8.9% | Check whether landing content matches what drove the click (ad/search mismatch), and page load speed. |
| JavaScript Errors | 218 | 3.1% | Worth checking browser/device breakdown to see if errors are concentrated on one platform. |

Recommendation rules:
- If sessions affected = 0, write "None — no occurrences in range."
- If nonzero, give one short, concrete, metric-appropriate recommendation (1 sentence). Don't speculate about specific root causes you can't see from aggregate counts alone.
- Don't add extra columns, extra rows, or prose analysis beyond the table and the one-line header note, unless the user asks for it.

### 6. Digging deeper (only if asked)

If — and only if — the user follows up asking to dig into a specific flagged issue (e.g. "which pages have the dead clicks," "break quick backs down by device"), use additional `query-analytics-dashboard` calls with narrower natural-language queries (filtered/broken down by page, browser, device, country, etc. as relevant) to get more detail. There is no recordings or heatmap tool available on this connector — don't imply you're pulling recordings; you're just querying the dashboard with more specific filters. Don't proactively offer this in the initial report — wait for the user to ask.

## Notes

- Today's date should come from the actual current date, not an assumption.
- If a metric has zero affected sessions, still include its row (with "None" recommendation) — don't drop it from the table.
- Don't call `visualize:show_widget` or any charting tool for this skill — the deliverable is the table only.
- Total 6 calls per report (well under typical daily quotas) — no need to warn the user about rate limits under normal use.
