---
name: gap-growth-goals-math
description: 'Builds the Sales & Marketing Goals slides for an Improve & Grow (I&G) Growth Action Plan deck: for each revenue stream, the math that turns the revenue goal into what marketing has to generate (inquiries, enrollments, bookings, orders, jobs, calls), shown on a slide with two stat cards, a math table with a highlighted goal column, a footnote and a takeaway line. Use whenever the user wants growth goals math, sales and marketing goals, "what we need to generate to hit the goal", lead or enrollment targets, or a goals slide for a GAP or proposal presentation. Every client and every revenue stream works differently, so the skill works out how this business sells with the user, proposes the math, asks for the inputs it needs, builds and verifies the numbers, then builds the slide. Output is .pptx slides built with ig-deck-style, ready to import into the I&G Google Slides deck.'
---

# GAP: Growth Goals Math

For each revenue stream, turn "grow revenue from X to Y" into the volume of demand marketing has to create, in the unit this client actually counts. The slide shows the arithmetic so the client can see where the number comes from and challenge the assumptions.

**There is no standard model.** Every client sells differently, and so does every stream inside one client. Your job is to work out the right math for this client with the user, ask for what you need, build it up transparently, verify it, and shape it for the slide. Slides use `ig-deck-style` (required). Voice rules come from `improve-and-grow-brand`.

## Principles
- **Start from this business, not from a past client.** Do not carry rates, capacity limits, segment structures or scenario columns from one client to another. The worked examples in `references/worked-examples.md` show different approaches for different businesses. They illustrate; they are not templates.
- **Agree on the chain before computing.** Say in plain words how you think the goal turns into demand, and let the user correct it.
- **Ask for inputs, do not guess them.** Offer defaults only as labeled assumptions the user can accept or change.
- **Verify every figure,** and say plainly which numbers are client reported, from data or assumed.

## What you produce
One **SALES & MARKETING GOALS** slide per revenue stream that has a growth goal and a role for marketing. Title: the stream's name. Two stat cards, a math table with a highlighted goal column, a footnote with the key assumptions, a takeaway line, and speaker notes. See `references/slide-anatomy.md`.

## Workflow

1. **Start from the revenue breakdown.** Read the Revenue Breakdown spec or slide (`gap-revenue-breakdown`) for current revenue, goal and volume per stream. If it does not exist, collect those three numbers per stream first so the slides never disagree.
2. **Learn how this business sells.** Ask only what you cannot find in the files: what counts as one sale, how customers arrive and what happens between interest and purchase, what repeats, what limits growth (capacity, staff, seasons), and whether the stream has types or segments that behave differently. `references/building-the-model.md` has the question list and the building blocks.
3. **Propose the chain in plain words.** One or two options, each a sentence or two ("goal revenue divided by value per job gives jobs; subtract repeat customers; divide by the close rate to get leads"). Confirm or adjust with the user.
4. **Ask for the inputs the chain needs.** One short message per stream: what you already have, what you can compute yourself, and exactly what is missing (`references/inputs-checklist.md`). If the user cannot supply something, say what you would assume and why, and carry it as a labeled assumption.
5. **Build the numbers up in code.** Where the data is not split the way the math needs (revenue by type, by segment), build it from what you do know and say how (`references/building-the-model.md`). Write a `checks` entry for every figure that goes on the slide.
6. **Sanity-check.** Does the goal fit any real limit (capacity, staff, season)? Is anything already full or unable to grow? Does the monthly target look plausible next to today's? Is one blended average hiding very different types? Say plainly if the goal does not fit or looks like a stretch, and show the number.
7. **Shape the slide** (`references/slide-anatomy.md`) and write the spec JSON. Give the user the headline and the assumptions in a few lines in chat.
8. **Build and look.** `python scripts/build_goals_slides.py spec.json "<out>.pptx" [--breakdown revenue-breakdown.json]`. Resolve every WARN. Render with `ig-deck-style/scripts/qa_render.py` and view each slide. Shorten text before shrinking it.
9. **Hand over.** Present the .pptx. List the assumptions the client must confirm and anything held flat or changed. Keep the spec as the record of the math.

## Rules
- **Never invent a rate.** Repeat, conversion, attrition and utilization rates come from the client, from data, or from an assumption you label and flag. Name who reported each in the footnote.
- **Verify every figure** with a `checks` formula. Round to the nearest whole number; use "about" for per-month or per-week rates.
- **Prefer year-free labels** ("Current", "Goal", or the client's words) so the slide stays valid.
- **Takeaway line:** "You need a plan to generate N {units} to reach the annual goal," in the unit the client counts. Adjust the wording when the answer is a mix.
- **Work with what you have.** If something cannot be confirmed, use the best labeled estimate and list what to confirm in the speaker notes. Do not stall the slide.
- One slide per stream, in the order of the Revenue Breakdown. Skip streams with no goal or no role for marketing unless the user wants them.
- Plain words, no em dashes (see `improve-and-grow-brand/references/copy-voice.md`).
