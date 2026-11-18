---
name: gap-competitor-seo-stats
description: 'Builds the Top N Competitor SEO Stats slide for an Improve & Grow (I&G) Growth Action Plan: pulls Ahrefs Site Explorer figures for the client and its direct competitors (organic visits, ranking keywords, ranking pages, keywords in the top 3, referring domains, backlinks, paid search keywords) and builds a comparison table with three takeaways. Use whenever the user wants a competitor SEO comparison, SEO stats, an Ahrefs overview of competitors, or the SEO stats slide for a GAP or proposal presentation. Works for any business and any number of revenue streams. Takes the competitor list from the share of voice analysis, the session, the Client Overview, or the user. Output is a .pptx slide per revenue stream (or one slide) built with ig-deck-style.'
---

# GAP: Competitor SEO Stats

Shows how the client's organic search footprint compares with its direct competitors, in numbers a client can read: visits, keywords, pages and links. Slides use `ig-deck-style` (required). Voice rules come from `improve-and-grow-brand`.

## Settings
- **Competitors.** Use, in this order: the `competitors.json` from `gap-competitor-share-of-voice`; competitors the user named in this session; the competitors in the ClickUp Client Overview; otherwise ask the user. Do not invent a list. If there is none yet, offer to run the share of voice analysis first. The number of competitors is the user's choice (a table fits up to six competitors plus the client).
- **Streams.** One slide per revenue stream when competitors differ by stream (use the `streams` field in `competitors.json`). One slide when the client has no streams.
- **Country.** The search market for the figures, usually the client's country. Confirm it.
- **Scope.** Each site is measured on its whole domain unless a competitor sits inside a larger site. If a competitor is a program or section of a bigger organization (for example a camp inside a community center's site), measure the section (`path` in `competitors.json`, prefix mode) or say that the figures cover the whole site.

## What the table shows
Client first (highlighted), then the competitors. Rows in this order: **Organic visits per month, Ranking keywords, Ranking pages, Keywords in the top 3, Referring domains, Backlinks, Paid search keywords.** Domain Rating is left out by default: it is a score clients cannot interpret and it prompts more questions than it answers. Add it only if the user asks (`--domain-rating`).

## Workflow
1. **Get the competitor list** (above) and confirm it with the user, with the client's domain.
2. **Pull the figures.** Follow `references/ahrefs-guide.md`: one `batch-analysis` call for all sites (client plus competitors) for visits, ranking keywords, keywords in the top 3, paid keywords, referring domains and backlinks; then `site-explorer-pages-history` per site for ranking pages (the latest monthly point). Save the batch result and a small `pages.json`.
3. **Assemble:** `python scripts/assemble_stats.py stats.json --batch batch.json --pages pages.json --competitors competitors.json --client-name "<Client>" --client-domain <domain> [--stream "<Stream>"] --country us --date <YYYY-MM-DD>`.
4. **Read the numbers** with the user's goal in mind (`references/slide-spec.md` has what to look for) and write three takeaways. Lead with what is true and useful: where the client leads, where competitors earn traffic and how, and what the client already has to build on.
5. **Build:** `python scripts/build_seo_stats_slides.py out.pptx --takeaways takeaways.json stats1.json [stats2.json ...]`. Every number in a takeaway is checked against the data. Render with `ig-deck-style/scripts/qa_render.py` and view each slide.
6. **Hand over.** Present the .pptx and give the date of the pull. Say if any competitor was measured on a section, and flag anything surprising.

## Rules
- **Use the same competitors as the share of voice slide** unless the user changes them. Do not mix lists across slides.
- **Never present Ahrefs traffic as exact.** It is an estimate. The source line says so.
- **Do not edit figures by hand.** The builder takes them from the pull. It warns on a takeaway number that is not in the data.
- **Look for the story behind the numbers** before writing takeaways: pages versus traffic, links versus traffic, who runs paid search and where the traffic comes from (`site-explorer-top-pages` for a competitor when the user wants depth).
- Plain words, no em dashes (see `improve-and-grow-brand/references/copy-voice.md`).
