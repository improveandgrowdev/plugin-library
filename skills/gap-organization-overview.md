---
name: gap-organization-overview
description: 'Builds the Organization Overview slide for an Improve & Grow (I&G) Growth Action Plan deck: who the client is and a description of what they offer, in the I&G deck style. Use whenever the user wants the organization overview, company overview, "about the client" or programs slide for a GAP or proposal presentation, even if they only say "do the first content slide". Works for any client: camps, retreat centers and conference centers, and other businesses. Uses the ClickUp Client Overview and the client''s website. It has no numbers on it; revenue, volume and capacity go on the Revenue Breakdown slide (gap-revenue-breakdown). Output is a .pptx slide built with ig-deck-style, ready to import into the I&G Google Slides deck.'
---

# GAP: Organization Overview

One slide that tells the audience who the client is and what they offer. The numbers (revenue, volume, capacity) live on the Revenue Breakdown slide that follows, built by `gap-revenue-breakdown`, so they are not repeated here. Slides use `ig-deck-style` (required); voice rules come from `improve-and-grow-brand`.

## What you produce

**ORGANIZATION OVERVIEW** slide. Title: the client's friendly name. Left: a rounded photo if one is available. Right: "Organization Overview:" with 3 to 4 fact bullets, then "Programs:" (or "Services:" for non-camps) with one descriptive bullet per program or service.

Field names and the JSON spec: `references/slide-spec.md`. Where each fact comes from: `references/source-map.md`. Read both before drafting.

## Workflow

1. **Gather without asking first.**
   - ClickUp: load the ClickUp tools if needed and read the client's Client Overview page (ask for the link only if you cannot find it). Take company facts, background, services overview and audience from it.
   - Website: fetch the site's home, About and program or service pages. Use them for program descriptions and any fact the Client Overview lacks. Say which facts came from the website.
   - Photo: you cannot download images from the web in the sandbox. Ask the user for one photo (grounds, building or lake) if none was supplied. Without one, build the slide without it and say so.
2. **Draft the spec.** Facts only from the sources. If a fact is missing, leave it out and list it as a gap. Use the client's own program names.
3. **Run the builder:** `python scripts/build_org_overview.py spec.json "<out>.pptx" [--photo photo.jpg]`. Resolve every WARN.
4. **Check by eye.** Render with `ig-deck-style/scripts/qa_render.py` and view the slide. Fix overflow by shortening bullets, not by shrinking text.
5. **Hand over.** Present the .pptx. In two or three sentences say which sources you used, what is missing, and anything inferred. The user imports the slide with Insert > Import slides.

## Rules

- **Overview bullets (3, at most 4):** ownership or affiliation, founded and where, setting and how they operate (year-round or seasonal). Short and factual.
- **Programs:** each bullet is the program name, a colon, and a plain description of what it is and who it is for. About 90 characters at most. Use the client's program names, in the order the client would list them.
- **No numbers that belong on the Revenue Breakdown slide:** no guest, session, event, site, donor, capacity or revenue counts, and no percentages. The founding year and the client's own descriptive numbers ("a four-day first taste to a full week") are fine. The builder warns on the rest.
- **General path (non-camps):** heading "Services:". Take the list from the Client Overview's Services Overview and the website's service pages, most important first.
- Plain words, no em dashes (see `improve-and-grow-brand/references/copy-voice.md`).
