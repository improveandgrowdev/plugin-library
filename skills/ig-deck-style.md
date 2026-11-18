---
name: ig-deck-style
description: Improve & Grow (I&G) slide style and slide builder. Use whenever the user wants slides, a slide deck, or a presentation in the I&G brand, including slides to paste into an existing I&G Google Slides deck, section or subsection break slides, KPI cards, bar charts, comparison tables, rating grids, takeaway cards, or summary slides. Other I&G skills (gap-*, brand-messaging-guide) call this skill for all slide styling. Produces .pptx files built on the I&G template so slides import into the branded deck with the look intact. Not for HTML or artifact slides.
---

# I&G Deck Style

The `improve-and-grow-brand` skill is the source of truth for the I&G brand: colors, type, signature elements, copy voice and writing rules. Read its `references/copy-voice.md` before writing slide copy. This skill adds the slide template, layouts and builder, and uses those brand values. The template's master and layouts use the brand tokens exactly; slide-only additions (rating pill colors, the gold highlight) are listed in `references/style-guide.md` and in the brand tokens under `slides`.

One source of truth for how I&G slides are built. It holds the brand template, the slide patterns, and a Python builder. Any skill or task that outputs I&G slides builds them here instead of defining its own style.

## What is in this skill

- `assets/ig-template.pptx`: the I&G master and layouts with no slides. The cream background, three-dot motif, route graphic, logo and page numbers all come from it. Never recolor or redraw them.
- `scripts/ig_deck.py`: the builder (python-pptx). Slide types and patterns are methods on `Deck` and `Slide`.
- `scripts/qa_render.py`: renders the deck to PNGs through LibreOffice, makes a contact sheet, and lints the text.
- `references/style-guide.md`: tokens, slide anatomy, layout rules, copy rules. Read it before designing a new slide type.
- `references/patterns.md`: every builder method with its arguments, limits and a short example. Read it before writing a build script.
- `examples/example_build.py`: builds one of every slide type. Copy from it.

`scripts/check_brand_tokens.py` compares this builder's colors with the brand skill's tokens and reports drift. Run it after changing either skill.

Requirements: `python-pptx`, `lxml`, `Pillow`, LibreOffice (`soffice`) and `pdftoppm` for rendering. Fonts are Fraunces (headings) and Open Sans (body), both free on Google Fonts. Install them for faithful renders. Google Slides has them built in.

## How to build slides

1. Decide the slide types needed. Use the standard types before inventing one (see `references/style-guide.md`).
2. Write a short Python script that imports the builder and creates the slides:

```python
import sys
sys.path.insert(0, "<path to this skill>/scripts")
from ig_deck import Deck
d = Deck()
s = d.content_slide("KEYWORD RESEARCH", "Overnight Guest Groups")   # kicker, title
s.kpi_row([("Keywords researched", "121"), ("Keyword categories", "9")])
s.notes("Speaker notes here")
d.save("/mnt/user-data/outputs/<name>.pptx")
```

3. Render and look at every slide: `python scripts/qa_render.py <file>.pptx <outdir>`, then view `slide-NN.png` (or `contact-sheet.png` for an overview). Fix overflow, overlap and cut-off text. The render uses LibreOffice, so spacing can differ slightly from Google Slides. Trust it for gross problems, not for pixel fit.
4. Fix everything the lint prints (em dashes, missing notes with `--notes-required`, notes over 4,000 characters).
5. Present the .pptx with the file tool. Tell the user how to bring the slides in: open the .pptx in Google Slides, or in the target deck use Insert > Import slides and pick the slides. The slides use the same master and layouts as the I&G deck, so they keep the look either way.

## Rules that always apply

- **Title is the subject, kicker is the analysis.** Content slides use the revenue stream, client or topic as the title and an ALL-CAPS kicker for the analysis (KEYWORD RESEARCH, TOP 5 COMPETITOR SHARE OF VOICE). The builder upper-cases kickers.
- **Section breaks:** major sections use `section_slide` (dark green, kicker like SECTION 2). Subsections, such as one per revenue stream, use `subsection_slide` (cream). A single-stream plan does not need subsection slides.
- **Native shapes only.** Cards, bars, pills and tables are editable shapes and text boxes. Do not paste images of charts.
- **One source line** on slides with data, naming the source, date and any rounding caveat. Speaker notes on analysis slides: what the slide shows, how to read it, talking points, caveats. Keep notes under 4,000 characters.
- **Ratings use words, not scores:** Strong, Solid, Mixed, Weak.
- **Copy:** follow `improve-and-grow-brand/references/copy-voice.md` (no em dashes, colons between names and descriptions, plain words, varied rhythm). Slide specifics: bold lead-ins on takeaways, findings and opportunity first, caveats in the source line or notes. Do not invent numbers; every figure traces to a source.
- **Do not edit the template's master or layouts** except through a deliberate change to this skill. If the deck's look changes, update `assets/ig-template.pptx` here (export the current deck, delete its slides) so every dependent skill follows.
