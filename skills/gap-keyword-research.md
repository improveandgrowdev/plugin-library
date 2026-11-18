---
name: "gap-keyword-research"
description: Guides keyword research for an Improve & Grow (I&G) Growth Action Plan from seed keywords to a categorized, approved keyword list and a Keyword Research slide. Use whenever the user wants to research keywords for a client, expand a seed list, find high-intent or conversion-intent search terms, pull or analyze Google Keyword Planner data, categorize keywords, or build the keyword research slide for a GAP presentation. Works for any business, not only camps. Starts by asking for a seed keyword list and the intent scope, expands it in Ahrefs Keywords Explorer, optionally mines keywords that competitors' websites rank for, lets the user approve additions, then helps with a Google Keyword Planner pull, builds the keyword workbook, and offers the slide. Then offers to set up the keywords in Ahrefs Rank Tracker, tagged by revenue stream, in the browser. Output is a keyword workbook (.xlsx) and a .pptx slide built with ig-deck-style.
---

# GAP: Keyword Research

You run a guided, approval-based keyword research process. The user approves every list before it grows or moves on. The goal is a categorized list of keywords with **high conversion intent** for each revenue stream, with Google Keyword Planner volumes, ready for the Keyword Research slide.

Slides use `ig-deck-style` (required). Voice rules come from `improve-and-grow-brand`. The logic is general: it must work for a plumber as well as a camp. The client's own nouns, places and categories come from a client config file (`examples/keyword-config.example.json`).

## What high intent means (the default target)

Keywords likely to come from someone ready to hire, book, buy, rent, register or ask for a quote. Look for:
- **Business or provider qualifiers:** services, company, contractor, firm, agency, consultant, venue, camp, retreat center, conference center, location, rental, and the client's own nouns.
- **Location qualifiers:** near me, local, nearby, in or around a place the client serves.
- **Transaction qualifiers:** book, reserve, rent, hire, quote, pricing, cost, availability, register.
- **Super-specific phrases** with no qualifier at all, such as "christian retreat centers" or "christian summer camp".

Informational terms (ideas, activities, how to, templates), jobs and careers, free, and other organizations' names are not high intent. Mixed-intent terms (for example "women's retreat", which a guest or an organizer might search) need a local or venue qualifier to count. Full rules and examples: `references/intent-rules.md`.

## Workflow

### 1. Start with two questions (only these)
1. **The seed keyword list**, per revenue stream. Ask for the revenue streams if you do not already know them (from the Revenue Breakdown or the ClickUp Client Overview).
2. **The intent scope:** only high conversion intent (the default), or other intent types too (informational, mixed)? If the user is unsure, recommend high intent only.
Then read the Client Overview (ClickUp) for the services, service area and driving distances, and set up the client config (`references/intent-rules.md`). Use `scripts/screen_keywords.py --derive-nouns seeds.txt` to suggest the client's business nouns, and confirm them with the user.

### 2. Expand the seeds in Ahrefs Keywords Explorer
Follow `references/ahrefs-guide.md`: pass the seeds to Matching Terms and Search Suggestions, screen the results with `screen_keywords.py`, and show the user the proposed additions per stream, with volume, Ahrefs intent flags and why each was kept. Show what the screen removed. The user approves, edits or rejects. Keep the approved list as the working list (the connector cannot create an Ahrefs keyword list, so give the user a paste-ready list if they want one in Ahrefs).

### 3. Review Matching Terms and Related Terms for the approved list
Run Matching Terms and Related Terms for the approved working list, screen again, and recommend additional terms to add. The user approves; add them. Offer another pass if the additions are likely to surface more (new nouns or modifiers). Repeat until the user says they are done.

### 3b. Optional: mine competitor keywords
Offer it once, after the Matching and Related Terms passes: "Do you also want to look at the keywords your competitors' sites rank for?" If yes, follow the competitor section of `references/ahrefs-guide.md`:
- **Get the competitors.** Use, in this order: competitors the user already named in this session; the competitors listed in the ClickUp Client Overview; otherwise ask the user to supply them. If the user has none, offer to suggest some from Ahrefs (organic competitors of the client's domain, organizations only, no directories or booking sites) and let the user confirm. Ask which revenue stream each competitor belongs to.
- **Pull each competitor's ranking keywords** from Ahrefs Site Explorer (organic keywords, non-branded, ranking in the top 50, a minimum volume), save each to a CSV, and describe the competitors in `competitors.json` (`examples/competitors.example.json`).
- **Screen and merge** with `scripts/merge_candidates.py`. It applies the same intent rules, drops each competitor's own brand terms, removes keywords already on the working list, and groups the rest by how many competitors rank for them.
- **Present the candidates** per stream: keyword, Ahrefs volume, which competitors rank, best position, tier and why. Keywords that two or more competitors rank for are the strongest signal. Show what was removed. The user approves, edits or rejects, and the approved terms join the working list.
Ahrefs volumes and traffic are for discovery only. They never go in the workbook; the approved keywords get their volumes from Keyword Planner like the rest.

### 4. Offer the Google Keyword Planner pull
When the user is done, ask whether they want Keyword Planner data. If yes, follow `references/keyword-planner-guide.md`: ask for the criteria, suggest locations per revenue stream from the service area (within what Keyword Planner allows), and give the user the keyword list to paste in. The user does the Keyword Planner work and uploads the export. You analyze it.

### 5. Screen, categorize and approve
For each stream run `scripts/prepare_stream.py` on the export. It reads the Planner export, de-duplicates, screens for intent, and assigns categories from the client config. Then:
- Show the user the categories, the keyword counts per category and the review list (keywords needing a keep or drop decision). Merge highly similar groups (for example Christian, church and Bible; marriage and couples) and put each keyword in one category: audience terms win over generic ones.
- Check the Planner data: how many keywords have a search volume and both top-of-page bids (low and high range). Tell the user how many were left blank by Keyword Planner and ask for a new export if the bid columns are missing. Never estimate or fill in missing values.
- Record decisions with `scripts/set_keep.py`.
- Build the workbook with `scripts/build_workbook.py` (one tab per revenue stream with the Keyword Planner search volume and top-of-page bids (low and high range) for every keyword, under Google's own headers, plus a Summary tab).
- Offer to read back anything the user wants changed.

### 6. Offer the slide
Offer the Keyword Research slide. Write `takeaways.json` (2 to 3 positive, specific takeaways per stream; see `references/slide-spec.md`), then run `scripts/build_keyword_slides.py workbook.xlsx out.pptx --takeaways takeaways.json --date "<export date>"`. Render with `ig-deck-style/scripts/qa_render.py`, view each slide, and present the file.

### 7. Offer Rank Tracker setup and the hand-offs
- **Rank Tracker:** `scripts/select_tracked.py` picks about 30 to 60 keywords per stream, tagged by stream. Offer to set up Ahrefs Rank Tracker with them. Follow `references/rank-tracker-setup.md`: ask which location to track (every time), look for an existing project and the plan's keyword room, get the user's yes on exactly what will be created, set up the project, keywords and tags in the browser (Claude in Chrome), then read it back through the connector and report the keyword count per tag. If the browser is not available, give the user the paste-ready list and the steps, and verify the same way afterward. The share-of-voice analysis relies on the tags, so the stream names must match exactly.
- **PPC themes:** the categories become the ad group themes.

## The slide in one paragraph
Kicker KEYWORD RESEARCH, title is the revenue stream. Four stat cards: **keyword categories, keywords researched, total monthly searches, and the share of search volume from high-intent keywords.** A solid bar chart of monthly searches by category and three positive takeaways. Every number is computed from the workbook. If the list was filtered to high intent only, the high-intent share is 100%; tell the user and ask whether to keep it or replace that card. Details: `references/slide-spec.md`.

## Rules
- **Never invent volumes, bids or keywords.** Volumes come from Google Keyword Planner (or Ahrefs for discovery only, labeled). Say which.
- **Rank Tracker setup changes the user's Ahrefs account.** Ask the location each time, say what will be created, wait for a yes, and confirm by reading it back before saying it is done.
- **Approval gates:** the seed list, each addition round, the categories and the final list are all approved by the user before you move on.
- **Show what you removed** and why, so the user can overrule the screen.
- **Competitors are optional.** The core process never needs a competitor list. Only build one if the user opts in to competitor keywords.
- **Totals are directional.** Google rounds volumes and close variants share a number. The slide footer says so. Do not add Ahrefs and Keyword Planner volumes together.
- **Positive framing on the slide:** findings and opportunity first. Put caveats in the footer and speaker notes.
- Plain words, no em dashes (see `improve-and-grow-brand/references/copy-voice.md`).
