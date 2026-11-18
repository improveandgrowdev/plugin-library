---
name: improve-and-grow-brand
description: "Improve & Grow (camp marketing agency) brand: colors, type, signature elements, copy voice and writing rules, plus the Camp Growth Roadmap content, logos, icons, photos and templates. This is the source of truth for how I&G looks and sounds in any output. Use whenever creating or editing anything for Improve & Grow, Camp Marketing Agency, the Camp Growth Roadmap (Plan, Build, Grow), the Growth Action Plan, or the Grow Your Camp audience of camp directors: slide decks, video presentations and narration scripts, eBooks and PDFs, HTML pages, proposals, documents, emails, or marketing copy. For PowerPoint or Google Slides decks built on the I&G template, also use the ig-deck-style skill."
---

# Improve & Grow: brand, voice, and presentation style

Improve & Grow (I&G) is a camp marketing agency in Lancaster, PA. Founder: Carl Lefever. Audience: camp and retreat center directors. The offer: a Growth Action Plan, built on the Camp Growth Roadmap (3 stages, 9 steps).

## How to use this skill
1. Read the file for the output type before building anything.
   - PowerPoint or Google Slides decks on the I&G template (Growth Action Plans, proposals, reports, client decks): use the `ig-deck-style` skill for layouts and the slide builder. Come back here for voice, tokens and assets.
   - Narrated or video presentations (the roadmap deck): `references/presentation-style.md` (also read `references/presentation-script.md` for the narration voice)
   - eBook, PDF, document, or HTML page: `references/brand-style.md` and `references/pdf-build-recipe.md`
   - Any written copy: `references/copy-voice.md`
   - Anything that mentions the roadmap, step names, results, or links: `references/roadmap-content.md`
2. Use the real assets in `assets/` (logo, icons, roadmap graphic, photos). Do not redraw or re-describe them.
3. Never invent statistics, quotes, or client facts. Use only `references/roadmap-content.md`, or ask.

## What lives where (source of truth)
- **This skill** owns the brand: color and type tokens, signature elements, copy voice and writing rules, assets, and the guidance for docs, HTML, PDFs and narrated decks.
- **`ig-deck-style`** owns the PowerPoint and Google Slides template, slide layouts and the builder. It uses the tokens and voice from here. If the two ever disagree on a brand rule, this skill wins; if they disagree on slide mechanics, `ig-deck-style` wins.
- **Other I&G skills** (`gap-*`, `brand-messaging-guide`) point here for voice and to `ig-deck-style` for slides. Do not copy brand rules into them.

## Slides template
The I&G Google Slides template (used by `ig-deck-style`) uses these brand tokens exactly: paper background, line borders, forest dark bands, ink and body text, green kickers. Slide-only values (rating pill colors, the gold highlight, table header and band) are in `references/brand-tokens.json` under `slides`. Change a brand token here and run `ig-deck-style/scripts/check_brand_tokens.py` to see what the template needs.

Reading decks (Growth Action Plans) are denser than narrated decks: text can run 24 to 26 px (9 to 10 pt) with a 24 px minimum. The 34 to 38 px body size and 40-word limit apply to narrated decks only.

## The rules that matter most
- **Colors:** green #039247, amber #ED9C21, red #DC2C27 (sparingly), forest #0B4A29, paper #FBFAF6, mint #EDF4EE, ink #333B36, body #434444. Blue is retired.
- **Type:** Fraunces (headings, weight 800, tight tracking) and Open Sans (body, labels, buttons). Nothing else.
- **Signature move:** a key heading word in green with a SOLID amber (#ED9C21) underline. Never a faded or semi-transparent underline (it renders pink).
- **Three-dot rule:** three dots (red, amber, green) above headings and as dividers.
- **Stage colors:** Plan is red, Build is amber, Grow is green. Always show the full red-amber-green set.
- **Cards:** white, rounded (14px in documents), thin border, a thick colored top border that carries meaning. No soft shadows and no gradients with transparency in PDFs or slides: some viewers (Google Drive) draw them as flat grey boxes or pink tints.
- **Text over photos:** never directly. Flatten a solid forest-green fade into the image, or use a solid panel.
- **Copy:** never use em dashes. Short plain words. Vary rhythm. No calendar years in long-shelf-life content. See `references/copy-voice.md`.
- **Names are locked:** step and stage names in `references/roadmap-content.md` must not be changed.
- **Call to action:** the primary button always reads "Let's Make a Plan".
- **Icons on slides:** inline SVG (transparent), never PNG, placed directly on the off-white slide with no card or border around them.
- **One idea per slide.** Camp directors are busy. Never crowd a slide.

## Files
- `references/`: the style, voice, content and recipe files above, plus `brand-tokens.json`
- `assets/`: `logos/`, `icons/` (9 step icons, SVG and PNG), `roadmap/`, `photos/`, `ebook-cover.jpg`
- `templates/slides/`: example slide files for each layout (asset placeholders look like `{{asset:name}}`)
- `templates/ebook/`: the print stylesheet used for the eBook PDF
- `scripts/`: `render_pdf.py` (HTML to PDF with Chromium) and `slides_generator_reference.py` (how the roadmap deck was generated)

## Not included (ask the user)
The full-color logo and any file not listed under `assets/`. Fonts load from Google Fonts: Fraunces and Open Sans.
