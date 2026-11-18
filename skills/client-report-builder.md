---
name: client-report-builder
description: Generates a client's monthly marketing progress report for Improve & Grow from that client's finalized Client Report Spec. Use whenever the user asks to build, generate, create, run, or update a monthly progress report for a specific client (e.g., "build August's report for [Client]," "run the report for [Client]"), or wants a previously-built report corrected to match the standard format. Pulls real data via ClickUp, Windsor.ai (GA4, Search Console, Google Ads, Meta, LinkedIn, Microsoft Ads, GBP), and LeadHub, staying faithful to the client's spec and the bundled Standard Report Structure, never inventing new metrics, comparisons, or structure. If no finalized spec exists yet, or an existing one still has open items, this skill stops and points to client-report-spec-builder instead of improvising. Always consult this skill for report-building requests even when phrasing is brief.
---

# Client Report Builder

**Skill version 1.2 — tracks Standard Report Structure v2.0 / Plan Tier Reference v1.0 (September 2026)**

This skill is the report-*generation* tool in a two-tool system. A separate process (not this skill) builds and finalizes each client's Report Spec through conversation with the strategist. This skill's only job is to execute that finalized spec faithfully against real, current data, every structural and comparison-logic decision has already been made; this skill reads and executes, it does not decide.

**If a report needs a judgment call the spec doesn't already answer, that's a sign the spec is incomplete.** Stop and flag it to the user rather than deciding unilaterally, don't quietly patch over a spec gap inside a single month's report.

Two reference files are bundled with this skill and should be consulted throughout:
- `references/standard-report-structure.md` — the exhaustive, canonical definition of every section, card, table, chart, and comparison rule, including the e-commerce variant. If a report disagrees with this doc, the report is wrong. **This file supersedes any prior "Global Process Reference" document**, it now contains all operational rules (comparison logic, tone, data-handling principles, the never-show-cost-or-hours rule, verification requirements) in one place.
- `references/plan-tier-reference.md` — what each PPC/SEO/Website/LeadHub plan tier actually includes, plus legacy tier-name mappings. Kept separate since pricing/tiers change on their own schedule.

**Updating this skill**: when the Standard Report Structure or Plan Tier Reference documents change, replace the corresponding file in `references/` and bump the version line above, then repackage and reinstall (delete the old installed skill, upload the new `.skill` file, keeping the same `name` field so it's recognized as an update, not a duplicate). These bundled files are a snapshot as of packaging time, they don't update themselves.

## Step 0: Preconditions

**Check whether a finalized Client Report Spec exists for this client at all.** For now, during testing, specs aren't yet fetched automatically, expect the user to provide the spec directly (attached, pasted, or referenced from earlier in the conversation/project). If the user hasn't provided one and doesn't point to where it is, ask rather than assuming it doesn't exist. *(Once specs are stored as ClickUp docs in each client's folder, this step will be updated to look them up there directly; until then, don't claim to search ClickUp for a spec, that lookup isn't built yet.)*

**If no spec is available at all**, stop, tell the user this client hasn't been set up for reporting, and direct them to the **client-report-spec-builder** skill first, that's the tool that produces this spec. Don't attempt to improvise one inline.

**If a spec is available, confirm its Open Items section is empty.** If it isn't, stop for the same reason, an incomplete spec still has unresolved decisions on it, and finishing those belongs to client-report-spec-builder, not to this skill. Tell the user what's still open and point them there to close it out, then come back.

Only proceed past this step once a spec is in hand and is fully resolved.

## Step 1: Pull Live Contract Status

Pull the client's current contract from ClickUp, never cached. Confirm Service Subscriptions still match the spec and the contract is still Active. If subscriptions have changed, stop, that's a spec change, not something to absorb into one month's report.

## Step 2: Pull Real Data, Section by Section

For every section the spec marks as included, pull exactly what `references/standard-report-structure.md` specifies for that section, no more, no less, no substitutions:

- **Overview**: sessions, the goal metric (count or revenue, per spec), total conversions, other conversions, e-commerce additions if applicable (revenue, transactions, AOV). Channel breakdown and conversion-type breakdown, each with MoM and YoY (YoY only where the spec confirms that specific metric's prior-year data is sound). Use the channel attribution basis (first-user or session) the spec specifies.
- **Product Breakdown** (e-commerce only): top 10-15 products by current-period revenue, matched carefully across periods, watch for near-duplicate product name variants representing the same item, and don't call a product discontinued without checking it's genuinely absent from the current period's full data.
- **PPC, per platform in the spec**: impressions, clicks, conversions, spend for the chart and cards (never conversion value or ROAS as the chart or cards); the full campaign table, adding a Conversion Value column only where that platform's data source actually exposes it. Every listed platform gets this in full, regardless of its current activity level, a dormant platform still needs its real historical chart and real current-period numbers (typically zero), never a text-only note in place of the real structure.
- **SEO**: Search Console for the metric cards (impressions, clicks, CTR, position) and the property table. GA4's Organic Search channel for the chart specifically, Search Console's own history is too short for a clean trailing-12-plus-prior-year view. GBP gets its own separate card row under its own subsection heading, never merged with the GSC row.
- **Website Support**: uptime rate, mobile and desktop performance scores (always two separate cards), updates applied, open vulnerabilities. UX friction data for the four real signals the spec's UX tool actually reports (don't assume a "JS Errors" row exists unless confirmed). Include incident or vulnerability detail tables only when there's real content for them that period.
- **LeadHub** (only if included): leads by source, pipeline activity (plus the stage-progression card if the spec marks pipeline-stage usage Active), email/text activity by type, social media activity if the tier includes it.
- **Specialist notes**: check the actual dates referenced inside each note, don't trust its filename or label, notes are frequently about the prior month regardless of what they're titled.

## Step 3: Apply Comparison Logic Exactly as Specified

Use the spec's stated values, don't re-derive them: Metric Card Comparison Basis governs what every card displays; Narrative Comparison Emphasis governs which comparison the commentary leans on (cards show their configured basis regardless); Prior-Year Data Soundness is checked per metric, show "N/A" honestly rather than a forced number, and drop unsound comparison columns from tables with an explanatory caption rather than filling them with dead cells.

## Step 4: Build Charts

Every chart uses the data source and line structure `standard-report-structure.md` specifies for that section: **trailing 12 months ending with the report period** (not calendar-year-to-date), with a **trailing 12 months prior year** dotted overlay when that history genuinely exists, applied per platform subsection individually in PPC. A shorter window with a clear caption note when full history doesn't exist. No gridlines. Never fabricate a data point to fill a gap.

## Step 5: Draft the Narrative

Headline (specific to this period's real finding, not generic). KPI summary (the "why," informed by specialist notes and known context). Callouts (Key Win, Worth Noting, New Idea) only when genuinely true, a section with no real win doesn't get one manufactured. Work Completed/Planned pulled from real ClickUp tasks; any confirmed unresolved open item gets a Work Planned line, not just a commentary mention. **Overview never gets a Work Completed/Planned section** (by design, see Standard Report Structure Section 5), it gets a single, self-contained Worth Noting callout instead when there's a cross-cutting finding. Never show cost, price, or hours anywhere, for any client, with no standard exception.

## Step 6: Handle Anomalies and Resolved History

Report anomalies with real, unaltered numbers, explain them in commentary, speculate on magnitude where useful, add investigating them to Work Planned. Never exclude or adjust underlying data. Don't resurface anything the spec's resolved-history section marks closed. If something newly discovered looks like a real problem, verify it against more than one data point and raise it with the strategist before it goes in the report, don't publish a damaging conclusion unilaterally.

## Step 7: Assemble the Report

Single self-contained HTML file: sidebar (nav links for every included section, Download PDF and Contact Strategist buttons), hero header, every included section in standard order, Discussion Items (3-5 items genuinely drawn from the report's own content, with its own Add-a-Discussion-Item button on that page), References (every field filled or explicitly marked absent), footer.

## Step 8: Mandatory Pre-Delivery Verification

Before presenting, diff the report section by section, table by table, column by column against `references/standard-report-structure.md` and against a prior completed reference report. Specifically confirm: every platform subsection has its full chart/cards/table; Overview's two breakdown tables are present with full comparison columns; SEO's GSC and GBP cards are separate rows under separate headings; no section's metric set replaces the standard instead of adding to it; no cost/price/hours appear anywhere; every callout present is genuine. This step is not optional, format drift has happened repeatedly in real builds and was only caught because someone checked.

## Step 9: Present for Review

Deliver the report with a short summary of what it found. This is a draft for the strategist. If a requested change is structural (a different metric, comparison basis, or standing exception), it belongs in the client's spec doc, not just in this one report, say so plainly rather than quietly absorbing the change into this month's output only.
