---
name: gap-competitor-share-of-voice
description: 'Builds the Top N Competitor Share of Voice slide for an Improve & Grow (I&G) Growth Action Plan: pulls share of voice from Ahrefs Rank Tracker, sets aside directories, review sites and booking platforms, picks the client''s direct competitors, and charts them against the client. Use whenever the user wants a share of voice analysis, a competitor search visibility comparison, to find who owns the search results for the client''s keywords, or the competitor list for a GAP or proposal presentation. Works for any business and any number of revenue streams: tags are optional, and the number of competitors is adjustable. Output is a .pptx slide per revenue stream (or one slide) built with ig-deck-style, plus a competitors list the SEO stats and website review skills reuse.'
---

# GAP: Competitor Share of Voice

Share of voice shows who owns the search results for the keywords that matter to the client. The slide compares the client with its direct competitors and shows how much of the results directories and booking sites take. Slides use `ig-deck-style` (required). Voice rules come from `improve-and-grow-brand`.

## Settings (ask once, at the start)
- **Number of competitors (N).** Default 5. Adjustable per client and per stream. Say that it is a slide-fit and workload convention, not a rule.
- **Tags (optional).** If the client's Rank Tracker keywords are tagged by revenue stream, run one pull and one slide per tag. If not, run one pull for the whole project and make one slide. Do not insist on tags.
- **Home region.** Which states or areas count as the client's market. Confirm it. It decides which organizations are direct competitors.
- **Named competitors.** Competitors the user mentioned in this session or the Client Overview lists (ClickUp). They are always shown, even if they do not make the top N, and they are marked as named.

## Workflow
1. **Find the project.** Use the Ahrefs connector (`management-projects`) to find the client's Rank Tracker project. Ask only if you cannot identify it. If there is no project yet, tell the user what to set up (the connector cannot create projects or tags) and offer the keyword list from `gap-keyword-research` (`select_tracked.py`).
2. **Look at what is tracked.** `management-project-keywords` shows tags, locations and the number of keywords. Count the keywords per tag for the source line. If several locations are tracked, confirm which one to use.
3. **Pull share of voice.** `rank-tracker-competitors-domains` with the project, device, date, and, when using tags, a tag filter. Pull enough rows (80 to 200) that organizations in the client's region appear after the directories. Details: `references/ahrefs-guide.md`. Save the result as JSON.
4. **Classify and select.** `python scripts/select_competitors.py ahrefs.json out.json --config config.json --regions regions.json --stream "<stream>" --tag "<tag>" --keywords <count> --device desktop --location "<location>" --date <YYYY-MM-DD> --limit-used <rows>`. Domains are sorted into the client, aggregators (directories, review sites, booking platforms, social, media) and organizations. Rules: `references/classification.md`.
5. **Settle the regions.** Organizations with no known region are listed. Check the top ones (their website, a search) and add each to `regions.json` (domain to state), starting with the highest share, until the top N in-region organizations are clear. Ask the user about any you cannot place. Re-run.
6. **Confirm the list with the user.** Show the competitors with their share, the named competitors, the largest set-asides (directories and others), the share the competitors hold of in-region organizations, and who was set aside as outside the region. The user approves, swaps or changes N.
7. **Write the takeaways** (2 to 3, `references/slide-spec.md`) and build: `python scripts/build_sov_slides.py out.pptx --takeaways takeaways.json stream1.json [stream2.json ...]`. Every percentage in a takeaway is checked against the data. Render with `ig-deck-style/scripts/qa_render.py` and view each slide.
8. **Save the competitors.** Run `select_competitors.py` with `--competitors-out competitors.json` so the SEO stats and website review skills use the same companies.
9. **Hand over.** Present the .pptx. State the date of the pull and note that small shares move day to day.

## Rules
- **Direct competitors are organizations that offer what the client offers in the client's market.** Directories, review sites, booking platforms, marketplaces, social networks, forums, reference sites and tourism or news media are not competitors. Report their combined share instead.
- **Never guess a region.** Check the site or ask. Do not include an organization whose location you have not confirmed.
- **Name the competitors the way the client would.** Use short display names in the config `names`. The builder warns when it had to derive a name from a domain.
- **Report the date, device and location** in the source line. Re-pull if the data is more than a few days old when the slide is presented, because rankings change.
- **Do not edit numbers by hand.** Takeaways cite figures from the data. The builder warns on a number that is not in it.
- Plain words, no em dashes (see `improve-and-grow-brand/references/copy-voice.md`).
