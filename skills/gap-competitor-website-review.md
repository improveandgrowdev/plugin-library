---
name: gap-competitor-website-review
description: 'Builds the Top N Competitor Website Review slide for an Improve & Grow (I&G) Growth Action Plan: reviews each direct competitor''s public website on a fixed rubric (User Experience, Content Quality/Depth, Technical SEO, Conversion), rates each category Strong, Solid, Mixed or Weak from recorded evidence, and builds a color-coded grid with three takeaways. Use whenever the user wants a competitor website review, a comparison of competitors'' sites, website ratings, or the website review slide for a GAP or proposal presentation. Works for any business and any number of revenue streams. Takes the competitor list from the share of voice analysis, the session, the Client Overview, or the user. Not the deep audit of the client''s own site. Output is a .pptx slide per revenue stream (or one slide) built with ig-deck-style.'
---

# GAP: Competitor Website Review

Shows how the competitors' websites compare in the four things that decide whether a site wins customers: how easy it is to use, how good the content is, how sound it is for search, and how well it turns visitors into inquiries. The goal is a fair, repeatable comparison that points to what the client can do better. Slides use `ig-deck-style` (required). Voice rules come from `improve-and-grow-brand`.

## Settings
- **Competitors.** Use, in this order: `competitors.json` from `gap-competitor-share-of-voice`; competitors the user named in this session; the competitors in the ClickUp Client Overview; otherwise ask. Use the same list as the share of voice and SEO stats slides. The number is the user's choice (the grid fits up to six).
- **Streams.** One slide per revenue stream when competitors differ by stream; one slide otherwise.
- **The client's own site is not rated here by default.** The client's site gets a deep audit separately. If the user wants the client in the grid, score it with the same rubric and mark its row.
- **Rubric.** `references/rubric.json`: four categories with five criteria each. Keep it as is for comparability across clients. Change it only if the user asks, and say so.

## Workflow
1. **Confirm the competitors** and the streams.
2. **Choose the pages to read** for each site: the home page, the page for the stream's main offering, the pricing or rates page, the booking or contact page, an about or trust page, and one deeper content page (FAQ, planning guide or blog). Three at least, five or six where the site has them.
3. **Read the pages** with web_fetch (not search snippets). For every criterion record what you saw and which page it came from. Use `references/evidence-guide.md` for what to look for. If a site cannot be read (blocked, rendered by script), say so, rate only what you could see, mark confidence "limited", and rate conservatively. If the user supplies PageSpeed or Lighthouse results, use them for the speed criterion. Use the ranking pages count from the SEO stats for the footprint criterion when you have it.
4. **Score every criterion 0, 1 or 2** for every site against the anchors in the rubric, in the evidence file (`examples/evidence.example.json` shows the format, with fictional sites). Score all sites in one pass, criterion by criterion, so the standard is the same.
5. **Rate.** `python scripts/rate_sites.py evidence.json rated.json`. It converts scores to Strong, Solid, Mixed or Weak (80%, 60%, 40% of the maximum) and flags missing scores, missing notes and too few pages. Check the result for sense. Show the user the grid with the strongest and weakest evidence per rating. If they disagree, change the score and the note, not the rating.
6. **Write three takeaways.** Typical: the best model to learn from (with a concrete fact such as published pricing or a booking path), the common gap across competitors (often technical SEO or pricing clarity), and the open ground the client can take. Positive and specific. Numbers or facts in a takeaway must come from the evidence notes.
7. **Build:** `python scripts/build_review_slides.py out.pptx --takeaways takeaways.json rated1.json [rated2.json ...]`. Render with `ig-deck-style/scripts/qa_render.py` and view each slide.
8. **Hand over.** Present the .pptx. Say how many pages were read per site, which sites had limited evidence, and that the review is directional.

## Rules
- **Evidence first.** Every score has a note naming what was seen and where. Never rate from memory, reputation or a search snippet.
- **Same standard for every site.** Score all sites on one criterion before moving to the next. Do not adjust a rating to make a story.
- **Say what you could not see.** Technical SEO from public pages is directional. A full technical audit needs a crawl; say so.
- **Fair language.** Describe what the site does, not what its owner should be ashamed of. Competitor names stay professional.
- **Date the review** (it is on the slide). Sites change.
- Plain words, no em dashes (see `improve-and-grow-brand/references/copy-voice.md`).
