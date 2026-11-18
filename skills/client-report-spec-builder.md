---
name: client-report-spec-builder
description: Builds or updates a client's Report Spec for Improve & Grow's monthly reporting system by looking up the ClickUp contract and service subscriptions, checking which ad/analytics platforms are actually connected and active, reading context docs (Client Overview, Brand Messaging Guide, Growth Action Plan, specialist notes), and confirming every comparison-logic decision with the strategist before writing a finalized spec. Use whenever the user wants to onboard a new client to reporting, update an existing client's spec after a real change (plan upgrade/downgrade, new data source, new exception), or asks to "build," "set up," or "update the spec/reporting" for a client. Counterpart to client-report-builder, which generates the monthly report from a finalized spec; this skill only produces that spec, never a report. Always use for onboarding or spec-update requests even when phrasing is brief, e.g. "let's get Acme set up for reporting" or "Acme upgraded their PPC plan, update their spec."
---

# Client Report Spec Builder

**Skill version 1.1 — tracks Standard Report Structure v2.0 / Plan Tier Reference v1.0 (September 2026)**

This skill is the spec-*building* tool in a two-tool system. It is exploratory and conversational: it looks things up, proposes answers, and does not stop until the strategist and the skill agree on every open decision. Its output is a finalized Client Report Spec doc. It never generates an actual monthly report, that's a separate skill (client-report-builder), which reads the spec this skill produces and executes it faithfully every month without making new structural decisions.

**A spec isn't done until its Open Items section is empty.** Don't hand off a partial spec, keep working the list with the user.

Three reference files are bundled with this skill:
- `references/client-report-spec-template.md` — the fillable per-client document this skill produces. Fill in every field; never leave a decision implicit.
- `references/standard-report-structure.md` — the canonical report structure. Use this to know exactly which per-client decisions the spec needs to capture (which sections apply, which platforms, which comparison settings, etc.). **This file supersedes any prior "Global Process Reference" document**, it now contains all operational rules in one place.
- `references/plan-tier-reference.md` — what each plan tier actually includes, plus legacy tier-name mappings, kept separate since it changes on its own schedule.

**Updating this skill**: when the Standard Report Structure or Plan Tier Reference documents change, replace the corresponding file in `references/` and bump the version line above, then repackage and reinstall (delete the old installed skill, upload the new `.skill` file, keeping the same `name` field so it's recognized as an update, not a duplicate).

## Before Starting

Confirm the client's name and check for an existing spec doc first. This process is used both for a brand-new client and for deliberately updating an existing one, don't start from a blank slate if a partial spec already exists.

## Step 1: Contract & Service Subscriptions

Search for and pull the client's contract record (expect duplicate/cloned records in ClickUp; find the real, current one with live custom fields). Note Status, Service Subscriptions (mapped to labels), Contract Type, Objective, and Client Lead/Sales Lead. Fill the spec's Assigned Strategist field from these, but flag it for the user's explicit confirmation, a contract field isn't automatically the right person for client communication. Flag any legacy tier name (one that no longer appears on the current pricing page) as needing verification rather than assuming a mapping. If a subscribed service looks wrong, ask, don't accept it silently.

## Step 2: Connected Platforms

Check every ad/analytics connector (GA4, Search Console, Google Ads, Meta, LinkedIn, Microsoft/Bing, GBP, LeadHub) for an account matching the client. For every platform found, check whether it has real recent activity, a connected platform can still be dormant, note it as such rather than omitting it. If the number of platforms found doesn't match what the subscribed plan tier would suggest, surface that discrepancy rather than silently building around whatever's connected. Check whether GA4 has real, populated e-commerce fields before treating this as (or ruling it out as) an e-commerce client.

## Step 3: Context Documents

Find and read the Client Overview, Brand Messaging Guide (if a versioned pair exists, use the newer one), and the current Growth Action Plan. The GAP is often not where you'd expect, check the Client Overview's own linked references first, and if that only leads to a sales/contract-signing document, say so and ask rather than assuming none exists. Not every client will have a current-year GAP, a long-term client continuing a prior year's scope with an updated goal is a legitimate, common state. Distill Brand Voice into the spec rather than copying the guide's full text.

## Step 4: Specialist Notes & Known Issues

Pull the most recent PPC, SEO, and Website Support specialist notes and check the actual dates referenced inside each, not just the filename. Look for standalone incident/issue-tracking docs; if something looks like a past incident, check whether the underlying tickets are actually closed before assuming it's current, and confirm with the user whether resolved history should be mentioned in reports at all (usually not). Never publish a conclusion about a potential problem, bring it to the user, sourced and specific, and let them decide how much (if any) belongs in ongoing reports.

## Step 5: Fill the Comparison Configuration

This is where judgment calls live, don't rush it:
- **Seasonality**: ask directly, don't infer from industry. The answer may not be a clean single category (e.g., "seasonal, but frequently overridden by external macro factors"), preserve that nuance.
- **Prior-year data soundness**: check per metric, not per client as a whole. A client can have valid YoY traffic history and invalid YoY conversion history (a tracking vendor switch) at the same time.
- **Narrative comparison emphasis**: derive from the two rows above using the decision rule in `standard-report-structure.md`, and write out the reasoning.
- **Metric Card Comparison Basis**: default to Both unless there's a real reason to simplify.
- **Channel Attribution Basis**: propose first-user as a reasonable default, but state it as needing confirmation, don't settle it silently.

## Step 6: Confirm Every Section's Configuration

Walk through the spec template's section checklist explicitly with the user: which sections apply, which PPC platforms and their state, LeadHub's tier-based subsections and pipeline-stage usage, GBP permissions, the confirmed UX tool. Don't leave any item unconfirmed.

## Step 7: Resolve, Don't Guess

The operating rule throughout: verify with real evidence before concluding something, and raise it with the user before writing it into the spec as settled fact. A spec built on an unconfirmed guess just moves the drift problem from report-building time to spec-building time.

## Step 8: Finalize

Confirm the Open Items section is empty. Confirm the spec doesn't restate anything already covered by `standard-report-structure.md` (which now includes the plan-tier data's cross-reference via `plan-tier-reference.md`), it should only contain what's genuinely specific to this client. The finalized spec is what the client-report-builder skill reads from every month; any future change gets written back into this spec, not just applied to a single month's report.
