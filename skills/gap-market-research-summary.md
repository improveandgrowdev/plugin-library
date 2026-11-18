---
name: gap-market-research-summary
description: 'Builds the Summary slide that closes the Market Research & Competitive Analysis section of an Improve & Grow (I&G) Growth Action Plan (GAP): for each revenue stream, three bullets covering demand (keyword research), competition (share of voice and SEO stats) and the opening competitor websites leave (website review). Reads the four analysis slides from the GAP deck or the slides built earlier, writes bullets from what they say, checks every figure against them, and builds one bullets slide in ig-deck-style with speaker notes. Use whenever the user wants the market research summary, the competitive analysis summary slide, "wrap up the research section", a one-slide recap of keywords and competitors, or the summary for a GAP or proposal, even if they only say "do the summary slide for market research". Any client, any number of revenue streams.'
---

# GAP: Market Research Summary

Pull the section together in one slide. The four analyses (keyword research, share of voice, SEO stats, website review) are done per revenue stream. This slide says, per stream, what the demand is, how thin the competition is, and what is open. It adds no new research: everything on it comes from those slides.

Slides use `ig-deck-style` (required, installed next to this skill). Voice and brand come from `improve-and-grow-brand` (`references/copy-voice.md`). Read `references/summary-guide.md` before writing bullets.

## Workflow

1. **Find the inputs.** Use, in this order: the GAP deck the user attached or named, then the .pptx files from the analysis skills built earlier in this session, then ask the user for the deck. If the analysis slides do not exist yet, say so and name the skills that build them (gap-keyword-research, gap-competitor-share-of-voice, gap-competitor-seo-stats, gap-competitor-website-review). Do not research anything new.
2. **Extract the facts.** `python scripts/extract_market_slides.py <deck.pptx> [more.pptx ...] facts.json --md facts.md`. Read `facts.md`. Check that every stream has all four slides and read the warnings: stream names that differ between slides, and competitor sets that differ between the share of voice, SEO stats and website review slides. These are real inconsistencies in the deck, so tell the user.
3. **Draft the bullets** per stream, in order: demand, competition, opening (`references/summary-guide.md`). Figures come from the facts. Frame as opportunities. Write one short line each.
4. **Check with the user in chat**, briefly: the bullets per stream, the stream names you used, and anything the warnings raised. Ask once whether they are right or what to change. If the user already asked for the slide, build it and show the result instead.
5. **Build.** Write `spec.json` (`references/slide-spec.md`), set `"reviewed": true`, then:
   - `python scripts/build_summary_slide.py spec.json "<out>.pptx" --facts facts.json`. Fix every ERROR. Fix each WARN or tell the user which you left and why. A figure the builder cannot find on the source slides is wrong or derived: fix it or declare it under `derived`.
   - `python <ig-deck-style>/scripts/qa_render.py "<out>.pptx" <dir> --notes-required`, then view the image.
6. **Hand over.** Send the .pptx. In two or three sentences say what each stream's summary leads with, any naming or consistency problem found in the deck, and that it imports with Insert > Import slides after the last stream's slides.

## Rules that matter most

- **Never invent a number or a claim.** Every figure and every statement about competitors comes from the four analyses. If a stream lacks one analysis, summarize what exists and say what is missing.
- **Opportunity framing.** Strong search demand, thin competitors and gaps on competitor sites are the story. Avoid critical or negative wording about the client.
- **One name per stream.** Use the name the analysis slides use unless the user says otherwise, and report the mismatch.
- **Client-facing copy:** plain words, no em dashes, colons between a name and its description, no calendar years, inquiries and bookings rather than "leads".
- **One stream is the normal case.** Then the slide has no stream heading and 3 to 5 bullets.
- Do not edit the ig-deck-style template or builder. Be concise with the user and ask one question at a time.

## Files
- `scripts/extract_market_slides.py` reads the analysis slides into `facts.json`; `scripts/build_summary_slide.py` builds and validates the slide.
- `references/summary-guide.md` (bullet patterns, rules, the Pilgrim Pines reference) and `references/slide-spec.md`.
- `examples/pilgrim-pines-summary-spec.json` is the reference spec.
