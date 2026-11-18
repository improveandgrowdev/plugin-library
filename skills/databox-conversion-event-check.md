---
name: databox-conversion-event-check
description: Use this skill whenever the user wants to check whether conversion events are still triggering for a Databox client, asks about "stale events," "events that haven't fired," "events not triggering," or wants a conversion freshness check (28-day and 90-day windows) on conversion data pulled through Databox — regardless of where the events originate (Google Analytics, Facebook pixel, Google Tag Manager, etc.). Always use this skill for requests like "check if any events stopped triggering," "has [event name] fired recently for [client]," or "give me the 28-day event health table for [client]" — even if the user doesn't say "Databox" or "GA4" explicitly, as long as they reference a client and conversion/event activity. Produces a plain markdown table (event, last triggered date, 28-day total, 90-day total, status) and an explicit flagged list of any events with zero hits in the last 28 days. Text/table output only — no chart.
---

# Databox Conversion Event Freshness Check

Checks every conversion event for a given Databox client and confirms whether each one has triggered at least once in the trailing 28 days, with a 90-day total alongside for context. Produces a plain markdown table and an explicit flagged list — no chart.

## When to use this

Trigger for requests like:
- "Has [event] stopped firing for [client]?"
- "Check if any conversion events haven't triggered in 28 days"
- "Give me an event health check for [client]"
- "Are all our conversion events still working?"

This checks the client's **Conversions metric broken down by event name** in Databox. The events themselves can originate from any tag or pixel that reports into the client's analytics (Google Analytics, Facebook pixel, GTM-fired events, phone tracking, etc.) — the check is source-agnostic from the user's perspective. Technically, the data is read from the client's Google Analytics 4 data source in Databox via the `GoogleAnalytics4@conversions` metric, since that's where all conversion events are consolidated. It is not a general-purpose metric checker.

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
- **90-day window**: `start_90 = today - 89 days`

The fetch spans the full 90-day window, which doubles as the event roster: Databox's dimension breakdown only returns event names that actually had hits in the queried range, so the roster is "every event that fired at least once in the last 90 days." An event with zero hits in the full 90 days won't appear at all — if the user needs to catch those, they'd have to compare against a known list of expected events.

### 5. One single `load_metric_data` call

Make exactly **one** call — this is the only data fetch this skill needs, by design, to minimize how often the tool's automatic widget rendering shows up:

```
load_metric_data(
  metric_key="GoogleAnalytics4@conversions",
  dimension="eventName",
  data_source_id=<from step 2>,
  start_date=start_90,   # today - 89 days
  end_date=today,
  granulation_time_unit=2   # daily
)
```

This single daily series spanning 90 days gives you everything needed:

- **Roster**: every event name that appears as a series in the result — all events that fired in the last 90 days.
- **Last triggered**: for each event's series, the most recent date with a non-null value.
- **28-day total**: for each event's series, sum the values for dates from `start_28` through `today`.
- **90-day total**: for each event's series, sum all values in the range.

**Important — filter the series before post-processing.** The API response may include extra series beyond the current-period daily breakdown: previous-period comparison series (same event names, older dates) and single-point aggregate-total series. Only use the current-period daily series — identify them as the series whose data points fall within the requested date range with daily granularity. Do not sum across duplicate series with the same event name or totals will double-count.

Do not make additional `load_metric_data` calls for this skill (no separate "totals" call, no separate per-window call) — one call, post-processed four ways, is sufficient and keeps the widget side-effect to a single occurrence.

Note: `load_metric_data` is a rendering tool and may display its own inline widget automatically as a side effect of being called — that's a property of the tool itself, not something this skill controls. Don't add commentary about that widget, don't call any separate charting/visualization tool on top of it, and don't describe trends visually in prose. The deliverable for this skill is text only: the table and flagged list below.

### 6. Build the output

Output is **text only** — a markdown table plus a flagged list. No chart, no visualization call, no narrated trend description.

**Table**, one row per event from the baseline roster, sorted by last-triggered date (most recent first):

| Event | Last Triggered | 28-Day Total | 90-Day Total | Status |
|---|---|---|---|---|
| purchase | Jun 18 | 378 | 1,102 | ✅ Active |
| request_retreat_info_packet_form | Jun 14 | 3 | 11 | ✅ Active |
| some_quiet_event | May 3 | 0 | 7 | 🚩 Quiet |

- Status = ✅ Active if 28-day total > 0.
- Status = 🚩 Quiet if 28-day total = 0 (it fired within the 90-day window but has been silent for the last 28 days). Show its actual last-triggered date from the data (e.g. "May 3").
- Events with zero hits across the entire 90 days won't appear in the data at all, so they can't be listed — note this limitation only if the user asks about a specific event that's missing from the table.

**Flagged list**: after the table, call out quiet events explicitly and plainly, e.g.:

> 🚩 **Flagged (no hits in last 28 days):** `some_quiet_event` (last fired May 3)

If nothing is flagged, say so clearly and positively (e.g. "All N tracked events have triggered within the last 28 days — nothing flagged.").

## Notes

- Today's date for window math should come from the actual current date, not an assumption.
- If the client has a very high-volume analytics property, the roster call may return many event names — that's fine, just include all of them in the table.
- Don't ask the user for the date range or window length — it's fixed at dual 28-day and 90-day windows per this skill's design. Do ask if the client/account name is ambiguous.
- **No chart.** Don't call `visualize:show_widget` or any other charting/visualization tool for this skill. The only required deliverable is the markdown table + flagged list.
- **One `load_metric_data` call only.** This skill is deliberately designed around a single tool call (90-day daily breakdown by `eventName`), post-processed for roster / 28-day total / 90-day total / last-triggered, specifically to minimize how often the tool's automatic widget side-effect appears. Don't split this into multiple calls even if it seems more "natural" to fetch the roster and the window totals separately.
- **If the 90-day query fails** (e.g. Databox returns a server error on longer ranges for some sources), fall back to a 28-day query, report the 28-day columns only, and tell the user the 90-day view was unavailable.
