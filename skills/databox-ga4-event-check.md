---
name: databox-ga4-event-check
description: Use this skill whenever the user wants to check whether GA4 conversion events are still triggering for a Databox client, asks about "stale events," "events that haven't fired," "events not triggering," or wants a 28-day freshness check on Google Analytics 4 conversions pulled through Databox. Always use this skill for requests like "check if any events stopped triggering," "has [event name] fired recently for [client]," or "give me the 28-day event health table for [client]" — even if the user doesn't say "Databox" or "GA4" explicitly, as long as they reference a client and conversion/event activity. Produces a plain markdown table (event, last triggered date, 28-day total, status) and an explicit flagged list of any events with zero hits in the last 28 days. Text/table output only — no chart.
---

# Databox GA4 Event Freshness Check

Checks every GA4 conversion event for a given Databox client and confirms whether each one has triggered at least once in the trailing 28 days. Produces a plain markdown table and an explicit "stale" list — no chart.

## When to use this

Trigger for requests like:
- "Has [event] stopped firing for [client]?"
- "Check if any GA4 events haven't triggered in 28 days"
- "Give me an event health check for [client]"
- "Are all our conversion events still working?"

This is specifically scoped to **GA4 Conversions broken down by `eventName`** (the `GoogleAnalytics4@conversions` metric). It is not a general-purpose metric checker.

## Required tools

`tool_search` for `"Databox"` first if the Databox tools aren't already loaded in context. You need:
- `Databox:list_accounts`
- `Databox:list_data_sources`
- `Databox:list_metrics`
- `Databox:load_metric_data`

## Workflow

### 1. Resolve the client to an account_id

Call `Databox:list_accounts`. Match the user's client name case-insensitively against the `name` field (substring match is fine — e.g. "[Client]" matches "[Client]"). If there's no match or more than one plausible match, list the candidates and ask the user to confirm rather than guessing.

### 2. Find the GA4 data source

Call `Databox:list_data_sources` with that `account_id`. Find the entry with `"type": "Google Analytics 4"`. Note its `id` — this is the `data_source_id` for all subsequent calls. If there is no GA4 source, or more than one, tell the user and stop (don't guess which one).

### 3. Confirm the metric exists

Call `Databox:list_metrics` with that `data_source_id` and confirm `GoogleAnalytics4@conversions` is present (it has an `eventName` dimension). If it's missing, tell the user this client's GA4 source doesn't expose a conversions metric and stop.

### 4. Compute date ranges

Let `today` = current date.
- **28-day window**: `start_28 = today - 27 days`
- **Baseline window** (for discovering events that exist but went silent): `start_baseline = today - 89 days`

The baseline window matters: Databox's dimension breakdown only returns event names that actually had hits in the queried range. An event that's gone fully stale in the last 28 days simply won't appear in a 28-day-only query — so you can't detect "zero hits" without first knowing the event exists. Querying a longer baseline gives you the full roster of events to check against.

### 5. One single `load_metric_data` call

Make exactly **one** call — this is the only data fetch this skill needs, by design, to minimize how often the tool's automatic widget rendering shows up:

```
load_metric_data(
  metric_key="GoogleAnalytics4@conversions",
  dimension="eventName",
  data_source_id=<from step 2>,
  start_date=start_baseline,
  end_date=today,
  granulation_time_unit=2   # daily
)
```

This single daily series spanning 90 days gives you everything needed:

- **Roster**: every event name that appears as a series in the result — these are all the events that have ever fired in the last 90 days.
- **Last triggered**: for each event's series, the most recent date with a non-null value.
- **28-day total**: for each event's series, sum the values for dates from `start_28` through `today` only (ignore the older days — they're there purely to establish the roster).

Do not make additional `load_metric_data` calls for this skill (no separate "totals" call, no separate "28-day only" call) — one call, post-processed three ways, is sufficient and keeps the widget side-effect to a single occurrence.

Note: `load_metric_data` is a rendering tool and may display its own inline widget automatically as a side effect of being called — that's a property of the tool itself, not something this skill controls. Don't add commentary about that widget, don't call any separate charting/visualization tool on top of it, and don't describe trends visually in prose. The deliverable for this skill is text only: the table and flagged list below.

### 6. Build the output

Output is **text only** — a markdown table plus a flagged list. No chart, no visualization call, no narrated trend description.

**Table**, one row per event from the baseline roster, sorted by last-triggered date (most recent first):

| Event | Last Triggered | 28-Day Total | Status |
|---|---|---|---|
| purchase | Jun 18 | 378 | ✅ Active |
| request_retreat_info_packet_form | Jun 14 | 3 | ✅ Active |
| some_dead_event | May 3 (outside window) | 0 | 🚩 Stale |

- Status = ✅ Active if 28-day total > 0.
- Status = 🚩 Stale if 28-day total = 0 (it was in the baseline roster but didn't fire in the last 28 days). For the "Last Triggered" column on stale rows, if you can tell from the baseline data roughly when it last fired, say so (e.g. "last seen ~50 days ago"); otherwise just say "not in last 28 days."

**Flagged stale list**: after the table, call out stale events explicitly and plainly, e.g.:

> 🚩 **Flagged as stale (no hits in 28 days):** `some_dead_event`

If nothing is stale, say so clearly and positively (e.g. "All N tracked events have triggered within the last 28 days — nothing flagged.").

## Notes

- Today's date for window math should come from the actual current date, not an assumption.
- If the client has a very high-volume GA4 property, the baseline roster call may return many event names — that's fine, just include all of them in the table.
- Don't ask the user for the date range or window length — it's fixed at 28 days per this skill's design. Do ask if the client/account name is ambiguous.
- **No chart.** Don't call `visualize:show_widget` or any other charting/visualization tool for this skill. The only required deliverable is the markdown table + flagged list.
- **One `load_metric_data` call only.** This skill is deliberately designed around a single tool call (90-day daily breakdown by `eventName`), post-processed for roster/total/last-triggered, specifically to minimize how often the tool's automatic widget side-effect appears. Don't split this into multiple calls even if it seems more "natural" to fetch the roster and the 28-day totals separately.
