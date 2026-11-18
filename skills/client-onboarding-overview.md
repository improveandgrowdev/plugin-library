---
name: "client-onboarding-overview"
description: Fill in a client's ClickUp "Client Overview" doc from their completed LeadHub/GoHighLevel onboarding survey, for Improve & Grow's retreat/camp clients, and add the Camp Growth Planner summary and capacity-by-revenue-stream table from the client's Base44 planner results. Use whenever the user says a client finished, submitted, or completed their onboarding form and wants it reflected in ClickUp, asks to "onboard" a client, "pull [client]'s onboarding info into their doc," "set up [client]'s client overview," or references the Onboarding Information folder on a LeadHub contact together with a Client Overview / doc / ClickUp page — even if they don't name LeadHub or ClickUp explicitly.
---

# Client Onboarding → ClickUp Client Overview

This skill takes a client's answers from the LeadHub (GoHighLevel) onboarding
survey and writes them into that client's "Client Overview" doc in ClickUp,
then supplements gaps with public research (the client's own website and a
brand search for listing profiles), clearly flagging anything that didn't
come from the survey itself.

The point of the mapping below is that Improve & Grow's onboarding survey and
the Client Overview doc template were built to mirror each other almost
field-for-field — the work is mostly precise lookup and transcription, not
judgment calls. Where a field genuinely has no answer, distinguish *asked but
left blank* ("Not supplied") from *no such field exists* ("N/A") — this
distinction is what makes the doc trustworthy to a strategist skimming it
later.

## Step 1: Find the client's LeadHub contact

1. Use `mcp__LeadHub_MCP__list_locations` to confirm the "Improve & Grow"
   location and get its `locationId`. (It has historically been
   `di2otrR8TUMY7kpKeinW`, but re-verify by name each run rather than
   hardcoding it — locations can be added or renamed.)
2. Search for the client's onboarding contact with
   `search-contacts-advanced`. The obvious filter field `"name"` is
   **invalid** and returns a 400 — use `firstNameLowerCase` and
   `lastNameLowerCase` with `contains` filters instead. If you only have a
   company/camp name and not a person's name, search on that instead, or ask
   the user who the onboarding contact is.
3. Pull that contact's full custom field values (id → value pairs).

### If the search returns more than one contact

Clients sometimes have several contacts on file (different staff members,
duplicate entries, an old contact plus a new one), and only one of them may
actually be the person who filled out the onboarding survey. Don't just take
the first result:

1. Pull custom field values for each candidate contact and check which ones
   have values filled in under the Onboarding Information folder (Step 2
   tells you how to isolate that folder).
2. If exactly one candidate has onboarding data filled in, use that one —
   no need to bother the user.
3. If more than one has onboarding data (e.g. two staff members separately
   completed parts of it), or the situation is otherwise ambiguous, stop and
   ask the user which contact is correct. Give them something to
   distinguish by — name, email, job title if available, and how complete
   each one's onboarding data looks — rather than just listing bare contact
   IDs.

## Step 2: Isolate the "Onboarding Information" folder

The contact record carries custom fields from many different folders
(sales/discovery-call notes, Roadmap Assessment scores, tech-access fields,
etc.) — only the fields filed under the **Onboarding Information** folder are
in scope here, per how the user originally framed this task ("specifically
look for the content in the onboarding folder").

1. Call `get-custom-fields` with `model: contact` for the location to get
   every field's `id`, `name`, and `parentId` (its containing folder).
2. Confirm the Onboarding Information folder's id with `get-custom-field` —
   it has been `mpp6ZnI9mcE3NJ6h9TpO` for Improve & Grow, but verify by
   reading the folder's `name` rather than assuming the id is stable across
   locations or over time.
3. Filter the contact's custom field values down to only the ones whose
   definition has that folder as its `parentId`. Discard everything else
   (score fields, discovery-call fields, persona/job-title fields, etc.) —
   those belong to other parts of the sales process, not this doc.

`references/field-mapping.md` has the full list of onboarding field names
paired with where each one lands in the Client Overview doc, built from a
real client run-through. Field *names* here are what to match on (not raw
field IDs, which differ per client/location) — read it before writing the
doc so you're not guessing at the mapping from scratch each time.

## Step 3: Find the client's Client Overview doc

Search ClickUp for the client by name (`clickup_search`, `asset_types:
["doc"]`), then use `clickup_get_document_pages` to locate the page named
"Client Overview" (or equivalent) inside that client's doc. **Read its
current content first** — you need the exact existing structure of the
sections you're *not* touching (see Step 5) so your rewrite doesn't lose or
reformat them.

## Step 4: Supplement with public research

For every field that comes back blank from the onboarding data, before
marking it "Not supplied," spend a bit of effort seeing whether it's
genuinely derivable from public sources — this is worth doing automatically,
not just on request:

- **Fetch the client's own website.** Search for it first if you don't have
  a fetchable URL yet (search results and prior fetches are the only URLs
  `web_fetch` will open). The homepage, contact page, and footer often
  confirm or add: hosting/CMS platform, a Google Tag Manager container ID
  (visible directly in the page source), social profile links, a fax number,
  role-specific emails, and policy details (e.g., "we don't do weddings")
  that sharpen "Services Not Provided" or "Lack of Fit Factors."
- **Run a brand search** for the client name plus each missing listing
  platform (Apple Maps, Bing Places, Pinterest, X/Twitter, TripAdvisor,
  nonprofit directories like GuideStar/Candid, etc.) to try to fill listing
  URLs the onboarding survey left blank.
- Apple Maps and Bing Places listing pages in particular usually won't
  surface a stable, crawlable URL through web search the way Google Business
  or Yelp do — don't fabricate one. If you can't confirm a real URL, say so
  in the note rather than guessing.
- Every piece of information that came from research rather than the
  onboarding survey gets a short note flagging that, per the formatting
  rules in Step 5.

### Verify before accepting a brand-search result

A brand search for listing profiles can easily surface a page for a
different organization that just happens to share part of the client's
name — this is a real risk with common words in a business name (camps,
retreat centers, and churches especially tend to collide on names like
"Pines," "Grace," "Fellowship," etc.). Before treating any brand-search
result as belonging to the client, open it and check its listed address
and/or phone number against the client's own (from the Company Facts
section you've already filled in). **At least one of the two — address or
phone — has to match** before you accept the profile as genuine. If neither
is visible on the result, or neither matches, leave it out rather than
risk attaching a wrong link to the client's record; it's fine to just not
find everything.

List every profile you do confirm this way — beyond the four standard
platforms (Google Business, Apple Maps, Bing Places, Yelp), things like
TripAdvisor, GuideStar/Candid, industry-specific directories, or a
platform-specific page like Vimeo — under **Other Listing Platforms** as
single-level sub-bullets, one per confirmed profile, e.g.:

```
*   **Other Listing Platforms**:
    *   TripAdvisor: https://www.tripadvisor.com/... _(verified — address matches)_
    *   GuideStar/Candid: https://www.guidestar.org/Profile/... _(verified — EIN/address matches)_
```

Keep this to exactly one level of nesting with a plain 2-to-3-space indent
(4+ spaces risks the code-block problem described in Step 5) and no other
nested structure inside it — this is the one place in
the doc where a sub-bullet is worth the risk described in Step 5, but don't
extend that pattern anywhere else in the doc, and re-read the page after
writing to confirm it actually rendered as a list and not a code block.

## Step 5: Write the content — formatting rules that matter

These two formatting mistakes silently break the doc in ClickUp, so treat
them as load-bearing, not stylistic:

1. **Keep notes as plain italic parentheticals, nothing else.** Format
   researched additions as `_(plain sentence describing where this came
   from)_` — italics only, no bold, no color, no callout box. Critically,
   **don't write a bare domain name** (e.g. `pilgrimpines.org`) inside a
   note — ClickUp auto-links bare domains into colored hyperlinks, which
   reads as unwanted "special formatting" even though you never asked for
   one. Refer to the source generically instead: "the client's website," "a
   brand search," "the client's website footer."
2. **Never nest bullets more than one level deep.** ClickUp's markdown
   parser treats 4+ leading spaces as an indented code block, which silently
   swallows whole nested sections (this happened with the Business Listings
   subsection in testing — most of its content vanished into a `plain` code
   fence on write). For anything that reads like nested data — e.g. four
   platforms (Google/Apple/Bing/Yelp) each with four sub-fields
   (URL/Verified/Admin Access/Manager) — flatten it to single-level bullets
   with a compound bold label instead:
   `*   **Apple Maps — Verified**: No`
   rather than a bullet for "Apple Maps" containing four nested bullets
   underneath it. Group related platforms with a bold section header line
   (`**Business Listings**`) rather than an outer bullet.

Fill in only the **"Client Onboarding Form Responses"**, the **"Camp Growth
Planner"** section (Step 5b, camp and retreat clients only), and the **"Other
Information Collected After Onboarding"** sections. Leave the earlier
Company Overview / Point of Contacts / Marketing Services / Key Client
Documents section exactly as it is — those need contract records and
HighLevel contact links, not onboarding survey data, unless the user asks
you to fill those too.

For any onboarding field with no answer, mark it:
- **"Not supplied"** if the question exists on the intake form (i.e. the
  field id is defined in the Onboarding Information folder) but the client
  left it blank.
- **"N/A"** if there's no corresponding onboarding field at all for that doc
  line (nothing to ask in the first place).

## Step 5b: Camp Growth Planner section (camp and retreat clients)

Every camp or retreat client gets a **"Camp Growth Planner"** section in the
Client Overview. Place it **after the Digital Marketing Inventory (the end of
the onboarding form responses) and before "Other Information Collected After
Onboarding"**, under its own heading. It has two parts: a capacity summary and
a revenue table by stream. Skip it for clients that are not camps or retreat
centers. If the client has not completed the planner, say so in the doc and
leave a one-line placeholder.

### Where the data comes from

The Camp Growth Planner is a Base44 app. Find it with
`mcp__Base44__list_user_apps` (look it up by name; don't hardcode the id), then
`mcp__Base44__query_entities` on the `Assessment` entity, filtering `camp_name`
with a case-insensitive regex on the camp's name. Each record has
`assessment_data` (what the client entered) and `report` (what the planner
calculated). **Use the `report` values rather than recomputing**, so the doc
matches the planner's own report. Re-pull the record whenever the client or the
user says the planner was updated, and re-check every figure. If more than one
record matches, use the completed one with the latest `submitted_at` and say
which you used.

### Capacity summary (single-level bullets)

- **Total bed capacity**: from the report's facility totals. Say whether lodging
  was entered by type or as a single aggregate, and whether seasonality or
  season dates were captured. The planner ignores the "seasonal" checkbox on a
  lodging row when lodging is entered as an aggregate, so go by what the report
  shows (seasonal bed capacity and season months), not the checkbox.
- **Largest meeting space**: name and capacity.
- **Limiting factor**: the report's binding constraint.
- **Overnight guest groups**: min and max group size, max simultaneous groups,
  and current bookings (events, weekend/midweek split, total guests).
- **Day guest groups**: min and max group size and current bookings.
- **Inquiry-to-booked rate**: from the report's funnel, if entered.

### Revenue table

One row per revenue stream the client entered, then Fundraising, then a bold
Totals row, with exactly these columns in this order:

`Revenue Stream | Available Spots | Current Capacity Utilization | Current Annual Revenue | Revenue Goal | Capacity Utilization at Goal | Target Capacity Utilization | Target Capacity Revenue`

| Column | Source |
|---|---|
| Available Spots | `report.streams[k].available` |
| Current Capacity Utilization | `report.streams[k].utilization`, whole percent |
| Current Annual Revenue | `report.revenue[k].current_year` |
| Revenue Goal | `report.revenue[k].year1_goal`, **exactly as the client entered it; never change goals** |
| Capacity Utilization at Goal | `report.projections[k][0].util`, whole percent |
| Target Capacity Utilization | the **high end** of the planner's capacity range (`report.capacityTargets.high`, currently 90%) for **every** capacity stream, including Day Guest Group Rentals, so the table matches the planner's report |
| Target Capacity Revenue | current annual revenue + `report.streams[k].rev90` (the report's "Potential at 90% capacity") |

**Fundraising** has no capacity: show `n/a` for Available Spots, Current
Capacity Utilization and Capacity Utilization at Goal. Current revenue and goal
come from the report like any other stream. Its Target Capacity Utilization is
the high end of the planner's fundraising benchmark
(`report.benchmarks.fundraising_pct`, currently 20%), and its Target Capacity
Revenue is that percentage of **operating revenue in the target column** (the
sum of the other streams' Target Capacity Revenue), because the planner defines
fundraising as a share of operating revenue. Show it as calculated even when it
comes out below current fundraising, and say so in the footnote.

**Totals**: sum Current Annual Revenue, Revenue Goal and Target Capacity
Revenue. Leave the other total cells blank.

**Footnotes** under the table, as plain single-level lines:
- If lodging was entered as an aggregate with no season dates, put an asterisk
  on the **Overnight Guest Group Rentals** row label and add a footnote that the
  seasonality of the lodging needs to be clarified with the client, because
  available spots have no seasonal adjustment and may be overstated.
- A `**` footnote on the Fundraising row explaining the target calculation with
  the actual numbers.
- A short italic note that Revenue Goal is as entered by the client, Capacity
  Utilization at Goal is the utilization needed to deliver that goal, and Target
  Capacity Utilization is the top of the planner's range.

Add a source line under the section heading in the usual note format: who
completed the planner and when (`submitted_at`).

### Check before writing

Re-verify the arithmetic (sums, and that target revenue ties to the planner's
report), and flag in chat anything the user should know: goals that look like
placeholders (for example +50% on every stream), revenue that disagrees with
what the client said on calls, and a fundraising share far outside the 10% to
20% benchmark. If the user asks to see the table first, show it in chat and
write it to ClickUp only after they approve.

## Step 6: Write to ClickUp

`clickup_update_document_page` replaces the **entire page**, not just the
part you changed — always pass back the full content, including the
sections you're leaving untouched from Step 3's read.

Write directly once the content is assembled; no need to show it in chat
first. If the write comes back with an approval-related error (e.g. "No
approval received"), that's a permission gate in the ClickUp connector
itself, not a problem with the content — simply retry the same write once.
If it fails again, tell the user plainly and ask them to check for an
approval prompt on their end rather than silently giving up.

A write (especially an `append`) can report a server error and still go
through. **Before retrying after any error, re-read the page** and check
whether the content landed, otherwise a section can end up on the page twice.
Prefer a single full-page replace over repeated appends.

After writing, it's worth re-reading the page once to confirm it rendered
the way you intended (bullets didn't collapse into code blocks, notes didn't
turn into colored links) before telling the user it's done.
