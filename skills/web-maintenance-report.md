---
name: "web-maintenance-report"
description: "Assemble the agency's monthly website maintenance report for a client site by pulling data from WP Umbrella (updates, vulnerabilities, uptime), Databox (analytics tracking health), Google PageSpeed Insights (performance, Core Web Vitals), and Microsoft Clarity (UX friction signals). Use this whenever the user asks for a \"monthly maintenance report,\" \"web maintenance report,\" \"client report,\" or \"maintenance recap\" for a site — even if they only name the client or domain and don't mention the four tools explicitly. Always use this skill instead of running one-off checks when the request is clearly aiming at the full monthly report format (performance, issues, updates, security, UX friction, recommendations), rather than a single isolated question like \"is the site down\" or \"any vulnerabilities on X.\"\n"
---

# Monthly Web Maintenance Report

Builds the agency's standard monthly maintenance report for one client site. Pulls
live data from four sources — WP Umbrella, Databox, Google PageSpeed Insights, and
Microsoft Clarity — and assembles it into one markdown report the user can paste into
ClickUp themselves.

This skill does **not** write anything back to ClickUp, WP Umbrella, Databox, or
Clarity. It's read-only: gather data, build the report, hand it to the user.

## Step 0: Identify the site

Get the client name or domain from the user's request (e.g. "buckrentals.com"). If
neither is given, ask before proceeding — don't guess which client they mean.

Derive a short name for fuzzy-matching across the four systems (e.g. `buckrentals.com`
→ `buckrentals`). Strip `www.`, `https://`, and trailing slashes wherever you compare
URLs below.

**Reporting period**: use the month the user names. If they don't name one, default to
the most recent *complete* calendar month (e.g. a report run on Oct 5 covers September).
This month goes in the report title and sets the date range for "Updates applied" in
Step 1g. Note the Unix timestamps for the first second of the month and the first second
of the following month — you'll need them there:

```bash
FROM=$(date -u -d "2026-09-01" +%s); TO=$(date -u -d "2026-10-01" +%s)
```

## Step 1: WP Umbrella — uptime, updates, vulnerabilities, PHP errors

Load the WP Umbrella tools with `tool_search` if not already in context. Resolve
authentication per `skills/umbrella/SKILL.md` section 1 (token from
`$CLAUDE_PLUGIN_OPTION_token`, `$WP_UMBRELLA_TOKEN`, or `~/.umbrella/token`).

### 1a — Find the project

```bash
curl -sS -H "Authorization: Bearer $TOKEN" \
  "https://public-api.wp-umbrella.com/projects?per_page=200" \
  | jq --arg n "<short name>" '.data[] | select((.name | test($n; "i")) or (.base_url | test($n; "i"))) | {id, name, base_url}'
```

Zero matches → tell the user this site isn't in WP Umbrella and ask how to proceed.
Multiple matches → list them and ask which one. Never guess.

### 1b — Pull the project detail

```bash
curl -sS -H "Authorization: Bearer $TOKEN" \
  "https://public-api.wp-umbrella.com/projects/<id>"
```

This gives you `is_currently_down`, `latest_downtime`, `warnings` (PHP version/security,
SSL, WP up-to-date), and `last_synchronization`. Note the sync date — if it's more than
a few days old, mention that in the report so the user knows the data might be stale.

**Service interruptions**: WP Umbrella's public API has no monthly downtime *count* —
only a current `is_currently_down` flag and `latest_downtime` timestamp. Report this
as a point-in-time proxy, not a true monthly total, e.g.:

> Currently down: No. Last recorded downtime: <date, or "none on record">.

Don't imply this is an exhaustive count of incidents during the month — it isn't, and
saying so plainly avoids the report overstating its own precision.

**Uptime rate**: the public API also has no uptime percentage. If the user supplied an
uptime % in their request (e.g. copied from the WP Umbrella dashboard's uptime monitor),
use it in the At a Glance table. Otherwise, write `Not available via API — add from WP
Umbrella dashboard` so the gap is obvious to whoever pastes the report into ClickUp.
Never estimate or calculate an uptime % from `latest_downtime` — a single timestamp
can't support a percentage, and a made-up 99.9% is worse than an honest blank.

### 1c — Full plugin and theme update lists

The summary on `/projects/<id>` only flags *that* a plugin needs an update, not the
version numbers. For the actual current → new version pairs, hit the dedicated
endpoints:

```bash
curl -sS -H "Authorization: Bearer $TOKEN" \
  "https://public-api.wp-umbrella.com/projects/<id>/plugins?per_page=200" \
  | jq '[.data[] | select(.need_update != {} and .need_update != null) | {name, current: .version, new: .need_update.new_version}]'

curl -sS -H "Authorization: Bearer $TOKEN" \
  "https://public-api.wp-umbrella.com/projects/<id>/themes?per_page=200" \
  | jq '[.data[] | select(.version != .latest_version) | {name, current: .version, new: .latest_version}]'
```

### 1d — Broken links

```bash
curl -sS -H "Authorization: Bearer $TOKEN" \
  "https://public-api.wp-umbrella.com/projects/<id>/broken-links?per_page=100&show_ignored=not_ignored"
```

Returns a paginated list of broken links and images detected on the site. Make a single
call without `status_filter` or `resource_type` to capture everything at once. If
`total` (from the pagination envelope) exceeds 100, page through with `page=2`,
`page=3`, etc. until you have the full count — you don't need the full detail list, just
the totals by category.

Keep the following in your working notes for the Broken Links section in Step 5:
- Count of `4xx` responses (client errors, e.g. 404s)
- Count of `5xx` responses (server errors)
- Count of `timeout` responses
- A brief pattern note: if many broken links share the same domain, path prefix, or
  resource type, note that so the summary can be concrete (e.g. "Most are 404s pointing
  to deleted product pages" or "Three timeouts all from the same external partner domain").

### 1e — Vulnerabilities

```bash
curl -sS -H "Authorization: Bearer $TOKEN" \
  "https://public-api.wp-umbrella.com/projects/<id>/vulnerabilities"
```

Returns `plugin_vulnerabilities`, `theme_vulnerabilities`, and `wordpress_vulnerabilities`,
each with a `cvss_score` per vulnerability. Keep the CVSS score in your working notes —
you'll need it for the Security section in Step 5.

### 1f — PHP errors

```bash
curl -sS -H "Authorization: Bearer $TOKEN" \
  "https://public-api.wp-umbrella.com/projects/<id>/issues?per_page=100" \
  | jq '[.data[] | {message, severity, source, count, last_seen}]'
```

Returns a deduplicated list of PHP errors, warnings, and notices recorded on the site —
each entry represents a unique error signature, not an individual occurrence. Key fields:

- `severity`: `"FATAL"` or `"MINOR"`
- `count`: how many times this error has fired.
- `last_seen`: timestamp of the most recent occurrence.
- `source`: the plugin or theme slug the error originated from (if identifiable).

Errors from the WP Umbrella plugin itself are excluded automatically by the API.

If the list is empty, note "No PHP errors recorded" in the report. If `last_seen` is
recent (within the last 30 days) and `count` is high, treat the error as actively
occurring — flag it clearly rather than burying it.

### 1g — Updates applied this month

Step 1c covers what's still *pending*; this step covers what was actually *done*
during the reporting month. Pull the task history for the date range from Step 0:

```bash
curl -sS -H "Authorization: Bearer $TOKEN" \
  "https://public-api.wp-umbrella.com/projects/<id>/tasks?per_page=100&from=$FROM&to=$TO" \
  | jq '[.data[] | select(.type == "UPDATE_PLUGIN" or .type == "UPDATE_THEME" or .type == "UPDATE_CORE")
         | {type, code, name: .entities.name, from: .entities.old_version, to: .entities.version, created_at}]'
```

Filter by type client-side (as above) rather than with the `types` query parameter —
it's an array param and its encoding isn't documented, so client-side filtering is the
reliable option. If the response has more pages (check `links.pagination.last`), page
through with `page=2`, `page=3`, etc.

Keep in your working notes:
- **Updates applied**: the count of tasks with `code == "success"`, split by type
  (plugins / themes / core). This is the headline number for the At a Glance table.
- The list of successful updates (name: old → new version) for the Updates section.
- Any update tasks whose `code` is not `"success"` — these are failed updates and belong
  in the Updates section and likely in Recommendations, since a failed update usually
  means the site is still on a version someone intended to replace.

If the same plugin was updated more than once in the month, count each successful task
— each one is a real update that was applied.

## Step 2: Analytics — GA4 conversion event health

Invoke the `databox-ga4-event-check` skill for this client. That skill handles all
Databox authentication, account matching, and metric querying on its own — follow its
workflow fully, using the client name/domain from Step 0 as the input.

The skill produces:
- A markdown table: **Event | Last Triggered | 28-Day Total | Status** — one row per
  GA4 conversion event seen in the last 90 days, sorted by last-triggered date.
- A flagged stale list of any events with zero hits in the last 28 days.

Carry both outputs forward into the report as-is. If the skill cannot find a matching
Databox account or GA4 source, use its stated reason as the content for the Analytics
section (e.g. "No GA4 source connected in Databox for this site") rather than an empty
table.

## Step 3: Google PageSpeed Insights — performance

Load the `pagespeed-insights` tools with `tool_search` if not already in context. Use
the site's homepage URL unless the user specifies a different page.

1. Call `pagespeed-insights:full_report(url, strategy="mobile")` and again with
   `strategy="desktop"`. `full_report` combines the Lighthouse lab score with CrUX field
   data, which is what you need for both the score and the Core Web Vitals pass/fail —
   no need for a separate `crux_summary` call on top of it.
2. From each report, record: the overall performance score (0–100), and whether Core
   Web Vitals (LCP, CLS, INP/FID) pass or fail per the CrUX field data.
3. **Only if** the report flags a meaningful number of render-blocking resources as a
   top opportunity (not just one minor one), follow up with
   `pagespeed-insights:get_render_blocking_details(url, strategy="mobile")` for the
   specific files to name in your notes. Skip this call entirely when render-blocking
   isn't a significant factor — it's a drill-down, not a default step.
4. Write 1–3 sentences of "Additional Notes" — the most actionable opportunity or two
   from the report (render-blocking, unoptimized images, unused JS, etc.), in plain
   language a non-technical client could understand.

## Step 4: Microsoft Clarity — user friction signals

Invoke the `ig-clarity-ux-friction-report` skill for this client. That skill handles
connector identification, sanity-checking the right site's data, and all six Clarity
queries on its own — follow its workflow fully.

The skill produces:
- A one-line header with the date range and total session count (e.g. "Based on 7,043
  sessions — Aug 1–30, 2026.")
- A markdown table covering Rage Clicks, Dead Clicks, Excessive Scrolling, Quick Backs,
  and JavaScript Errors.

Carry both outputs forward into the report as-is, but use these column names:
**Name | Sessions | Effected Sessions | Recommendations**.

If no Clarity connector exists for this client, use the skill's stated reason as the
content for the User Friction section (e.g. "No Clarity connector found for this site.")
rather than an empty table.

**Note on JavaScript Errors**: Clarity tracks browser-side JS errors (sessions where a
JS error fired in the browser). These are separate from the server-side PHP errors in
Step 1e and appear only in the User Friction section. If the JS error % is notably high
(e.g. > 5% of sessions), call it out in Recommendations.

## Step 5: Build the report

Use this exact structure. Keep it tight — bullet points, not paragraphs, except where
noted.

```markdown
# Monthly Web Maintenance Report — <Client/Site> — <Month Year>

## At a Glance
| Metric | Value |
|---|---|
| Uptime rate | <N.NN% if the user supplied it, else "Not available via API — add from WP Umbrella dashboard"> |
| Mobile PSI performance | <N>/100 |
| Desktop PSI performance | <N>/100 |
| Updates applied | <N> (<P> plugins, <T> themes, <C> core) |
| Open vulnerabilities | <N> (highest CVSS <x.x>) — or "0" |

## Uptime & Performance
- Last known downtime: <date, or "None on record" — framed as a point-in-time snapshot, not a monthly count>
- PageSpeed Insights
  - Score: Mobile <N>/100, Desktop <N>/100
  - Core Web Vitals: <Pass / Fail — name which metric(s) failed if any>
  - Additional Notes: <1–3 sentences, plain language>

## Analytics
<Paste the full output from the databox-ga4-event-check skill here: the markdown table
(Event | Last Triggered | 28-Day Total | Status) followed by the flagged stale list.>
<If no GA4 source / no conversions metric: a plain one-line note explaining why, instead of an empty table.>

## User Friction
<Paste the full output from the ig-clarity-ux-friction-report skill here: the one-line
session header followed by the markdown table (Name | Sessions | Effected Sessions | Recommendations).>
<If no Clarity connector found for this site: "No Clarity connector found for this site." instead of the table.>

## PHP Errors
<Summary counts grouped by source (plugin/theme slug or "core") and severity, e.g.:>
- core: 1 fatal, 0 minor
- woocommerce: 0 fatal, 3 minor
- contact-form-7: 0 fatal, 1 minor

<If no PHP errors: "No PHP errors recorded." instead of the list.>

## Updates
- Applied this month:
  - <Plugin/theme/core name>: <old> → <new>
  - (repeat per successful update, or "No updates applied this month")
- Failed updates: <name: old → attempted version, or omit this line if none failed>
- Plugin updates pending:
  - <Plugin name>: <current> → <new>
  - (repeat per plugin, or "No plugin updates pending" if none)
- Theme updates pending:
  - <Theme name>: <current> → <new>
  - (or "No theme updates pending")

## Security
- Plugin vulnerabilities: <list with CVSS, or "None found">
- WordPress core vulnerabilities: <list with CVSS, or "None found">
- Theme vulnerabilities: <list with CVSS, or "None found">

## Broken Links
- 4xx: <N>
- 5xx: <N>
- Timeout: <N>
<One or two sentences summarizing the pattern, if any — e.g. "Most 404s point to deleted
product pages" or "The three timeouts all come from the same external partner domain."
If no broken links at all: "No broken links detected." instead of the counts.>

## Recommendations
- Quick wins (< 1 hour each), 2–3 of them:
  1. <what + why + how>
  2. <what + why + how>
- Bigger project (2–3 hours):
  - <what + why + how, scoped enough that someone could pick it up and start>
```

### Filling in At a Glance

This table is what the client reads first, often the only thing they read, so every
number in it must match the detail sections below it exactly:

- **Mobile / Desktop PSI** — the same scores as in Uptime & Performance (Step 3).
- **Updates applied** — the count of successful update tasks from Step 1g, matching the
  number of lines under "Applied this month." Failed updates are not counted here.
- **Open vulnerabilities** — the total across plugin, theme, and WordPress core
  vulnerabilities from Step 1e, matching the Security section. Include the highest CVSS
  score so a reader can tell "3 low-severity issues" from "1 critical" at a glance.
- If a source failed to load (e.g. PSI timed out), write "Unavailable" in that row
  rather than leaving it blank or dropping the row — keep all five rows every time so
  month-to-month reports line up.

### Writing good Recommendations

Base these on what you actually found this run — don't invent generic filler. This is
also where any specific errors worth calling out by name belong: the PHP Errors section
only shows grouped counts, so if a particular error is firing frequently and recently, a
specific vulnerability has a high CVSS, or the Clarity JS error % is notably high, name
it here with enough context to act on.

Good quick-win candidates: a single plugin update with no dependencies, fixing one
PHP-version warning, a render-blocking resource with a clear fix, or a low Clarity
metric (e.g. dead clicks on a single page) that's easy to reproduce and fix. Good
bigger-project candidates: clearing a theme update plus its dependent plugin chain, a
CVSS ≥ 7.0 vulnerability that needs careful testing before patching, a Core Web Vitals
failure that needs real performance work, or a high Clarity Quick Backs or Rage Clicks
rate pointing to a content/UX mismatch that needs investigation.

## Step 6: Hand off

Present the full report in chat as markdown. Don't post it anywhere — the user pastes
it into ClickUp themselves. If anything was unavailable (no WP Umbrella project found,
no analytics source, PSI failed to load, no Clarity connector for this site), say so
plainly in the relevant section instead of omitting it silently — a gap the client's
account manager needs to know about is more useful than a clean-looking report with a
quiet blank spot.
