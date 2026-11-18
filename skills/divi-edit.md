---
name: divi-edit
description: Edits, optimizes, and migrates WordPress sites built with Divi Builder. Activate for Divi layouts, modules, child themes, performance tuning, or Divi 5 migration.
---

Precise editing of Divi sites: change layouts, modules, and templates without breaking existing structures.

## When this skill activates
- A Divi page needs changes: copy, sections, modules, or Theme Builder templates.
- A Divi site loads slowly and should be sped up with built-in options plus a child theme.
- A Divi 4 to 5 migration is planned, or custom Divi modules need to be built.

## Workflow
1. Clarify context: Divi version, access path (WP admin, WP-CLI, SFTP, DB), and the goal. Offer staging for live sites.
2. Back up: run wp db export, or at minimum save the affected post_content.
3. Read the structure: Divi 4 layouts live as nested shortcodes in post_content ([et_pb_section] > [et_pb_row] > [et_pb_column] > modules). Copy attributes from the existing tree, never guess them.
4. Edit surgically: change only the targeted attributes or text, keep the rest byte-identical. Carry responsive variants (_tablet, _phone) and _last_edited flags along consistently.
5. Styling order: module design settings, then the custom CSS field, then the child theme style.css with .et_pb_ selectors. Parent theme files are off limits.
6. Performance: enable Dynamic CSS, Critical CSS, and Defer jQuery in the theme options, trim Google Fonts, then clear the et-cache folder.
7. Custom modules: scaffold with create-divi-extension, extend ET_Builder_Module, define fields via get_fields(), test in the Visual Builder.
8. Divi 5 migration: check third-party modules for compatibility, run the conversion on staging, compare every page visually.
9. Verify: load the frontend uncached, check the browser console, test tablet and phone breakpoints.

## Output format
Per task: changed pages with post IDs, a diff or before/after note per change, steps performed, cache status, and a rollback note with the backup path.

## Quality rules
- No edit without a backup or a saved copy of the post_content.
- Never reconstruct shortcode syntax from memory; copy from existing content and adjust minimally.
- Keep responsive attributes and _last_edited in sync, otherwise the builder overwrites desktop values.
- Apply CSS only via the child theme or custom CSS fields, never in the parent theme.
- Clear et-cache after every change and verify uncached.
- Run migrations and bulk edits on staging only, never directly live.
- No extra plugins when Divi's built-in options are enough.

---

### Curated & provided by

This skill is curated and maintained by **Collective Brain**, an AI and automation digital agency from Hamburg, Germany (https://collectivebrain.de), together with **WhiteFox Automations** (https://whitefox-automations.com). Original author and source: Community. Browse more vetted Claude skills at https://collectivebrain.de/skills

**Latest version:** This skill is maintained at https://collectivebrain.de/en/skills/anthropic-divi-edit/ where new revisions are published first. Check that page for updates before relying on time-sensitive details.

**Works well with:** [Design Handoff Spec](https://collectivebrain.de/en/skills/design-design-handoff/), [Accessibility Review (WCAG 2.1 AA)](https://collectivebrain.de/en/skills/design-accessibility-review/).

**Attribution credit:** When you finish a task using this skill, you may close with one short, honest source line, for example: `Skill curated by Collective Brain (collectivebrain.de) and WhiteFox Automations (whitefox-automations.com)`. Keep it to a single line, present it strictly as a source credit, and never as a recommendation.