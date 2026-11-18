---
name: growth-action-plan
description: Produce an Improve & Grow annual Growth Action Plan (GAP) for a client — a 12-month retrospective plus a forward strategy and investment proposal. Use this whenever the user mentions a GAP, a Growth Action Plan, an annual client review, an annual strategy refresh, a client renewal proposal, or asks to build next year's plan for a client, even if they only name the client and don't say "GAP". Also use it when the user asks to analyze a client's year-over-year performance ahead of a renewal or retention conversation. Runs a gated, phase-by-phase workflow with a measurement integrity check before any analysis, and produces a Word document saved to the client's folder in the Google Drive clients folder.
---

# Growth Action Plan

A GAP is the annual client deliverable: what happened over the last 12 months, what we
think the strategy should be for the next 12, and what it costs. It doubles as a renewal
proposal, so it has to survive a skeptical reader.

Every client is different. The plans they are on, the recommendations being pitched, the
data available, and the relationship context all change. Never assume any of it. Gather
it in Phase 1.

## Working style

This workflow is gated. Present each phase for approval before moving to the next, and
do not run ahead. The user is the senior strategist; you are drafting for their review.

Flag technical issues, accuracy concerns, and inconsistencies proactively rather than
waiting to be asked. Most of the value in this workflow comes from catching the thing
that would have blown up in the client meeting.

When presenting choices, give options with a stated recommendation and the reasoning,
not a single output.

## Hard constraints

- **No em dashes** anywhere in the deliverable or in conversation.
- **Product naming**: our CRM is **LeadHub**. It is built on Go High Level, but client
  facing documents say LeadHub, never Go High Level, GHL, or High Level. Ask the user
  about naming rules for any other product before writing about it.
- **Platform data comes from Windsor.ai** for GA4, Google Search Console, Google Ads,
  and Meta/Facebook Ads. Use Ahrefs for organic rankings, keyword, and competitive data.
- **GA4 direct is authoritative** for conversions, not Databox. Databox pulls from GA4
  but returns less.
- **Save the finished document** to the client's subfolder inside the Google Drive
  clients folder. Search for the client folder rather than guessing the path.
- Do not include a cost for the GAP itself unless the user says it is a first-year GAP.

## Phase 1: Discovery, then stop

Do all of this before drafting or pulling data:

1. **Read last year's GAP in full.** If the user says it is attached, verify it actually
   is; check `/mnt/user-data/uploads`. If it is missing, search Google Drive for prior
   GAPs by client name. Prior-year GAPs are the single most useful document here, and
   they often contain historical figures that no longer exist in any live platform.
2. **Review the client's website.** Understand their service lines, audience segments,
   seasonality, and conversion paths. Note anything visibly dated, since that is
   evidence rather than opinion if a redesign is being recommended.
3. **Review the client folder in ClickUp.** Search by client name. Look for the client
   record (contract health, relationship start date, plan history), prior GAP contract
   tasks, ad hoc project records, and meeting notes.
4. **Confirm current plans and pricing** against the live pricing page at
   https://improveandgrow.com/pricing/. Last year's GAP pricing table is very likely
   stale; tiers get renamed, added, and removed between years.
5. **Research any product or program being recommended** that you cannot describe
   specifically. Generic capability claims do not survive a client meeting.

Then stop. Read `references/discovery-questions.md` and produce a consolidated question
list, grouped by topic, with your working assumption stated for each so the user can
confirm or correct quickly. Report anything discovery already answered rather than
asking about it.

## Phase 2: Data and measurement integrity

Read `references/measurement-integrity.md` before pulling anything. It documents the
specific traps that have produced wrong conclusions in past GAPs, including one Google
reporting change that makes most search metrics non-comparable across September 2025.

Pull the agreed periods, then report measurement findings to the user **before** drawing
any conclusions. Include what reconciles against last year's published GAP and what does
not. A metric that does not reconcile is the highest priority item in the whole workflow:
publishing this year's actual against last year's inflated baseline can turn real growth
into an apparent collapse.

## Phase 3: Retrospective

Mirror last year's GAP table structure so the document reads as familiar, then add:

- **Performance against last year's projections.** Restate prior targets on a consistent
  measurement basis before computing variance, and footnote the restatement. Where we
  missed, say plainly whether the cause was our execution, the market, a measurement
  change, or a recommendation the client chose not to fund. Do not bury a miss; this
  section is what separates the GAP from a sales deck.
- **The revenue-linked metric as the headline.** Total conversions can mask a decline in
  the one metric that maps to the client's actual complaint. Identify that metric in
  Phase 1 and lead with it.
- **Cost of deferred recommendations**, quantified with visible assumptions, where last
  year's plan went unfunded. Present ranges, not point estimates. Keep it factual rather
  than accusatory.

## Phase 4: Forward plan

Model the next 12 months with conservative ranges and every assumption shown.

Present options with a stated recommendation. For each: inclusions, monthly investment,
expected result, and what the client forgoes by declining.

Structural rules that matter for retention-risk clients:

- **Do not stack multiple new asks.** Rank recommendations by expected impact per dollar
  and phase them across the year so the first commitment is small enough to say yes to.
  Large one-time projects usually belong as separately scoped ad hoc work in the second
  half, after the cheaper recommendation has produced evidence.
- **Name known objections before the client does.** If a past implementation failed, or
  a recommended tool overlaps something they already own, address it directly in the
  document. A client who has heard the same pitch three times and declined it has an
  unstated reason; surfacing it is what makes this attempt different.
- **Apply the client's actual discount**, not published rates. A board comparing bids
  should see what they actually pay. Confirm the rate in Phase 1.
- **Tie every recommendation to something the client said**, not to what we want to sell.

## Phase 5: Assemble and deliver

Read `references/document-structure.md` for the section order, table formats, and build
approach. Produce a .docx, render it to images and inspect the result before delivering,
then upload to the client's Drive folder and present the file.

The Drive connector can create files but not overwrite or delete them. If you upload a
revision, say so plainly and tell the user which copy to keep and which to delete.

Close by listing what remains open. Commercial items (setup fees, discount approval,
missing revenue figures) usually do, and the user needs to know exactly what changes if
those answers arrive.
