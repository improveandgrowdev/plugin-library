# Plugin Library

Internal landing page for the WordPress plugins and Claude skills built by the Improve & Grow team, published with GitHub Pages.

- **Edit the list:** update the `PLUGINS` array in `index.html` (name, type, version, release zip name, features, requirements).
- **New plugin release:** build the zip, attach it to a `v<version>` GitHub Release in the plugin's repo, then bump `version` and `zip` here.
- Downloads link to release assets in private repos, so they only work for people signed in to GitHub with access.
- **Skill files:** `skills/<command>.md` are copies of each skill's SKILL.md shown in the page's viewer. Client names and ClickUp IDs are redacted in these copies because this site is public; re-redact when refreshing them.
