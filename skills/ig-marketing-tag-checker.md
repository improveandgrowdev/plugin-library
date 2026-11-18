---
name: ig-marketing-tag-checker
description: Checks a website's homepage for marketing, analytics, and advertising tracking tags (Google Analytics, Google Tag Manager, Meta/Facebook Pixel, LinkedIn Insight Tag, TikTok Pixel, Hotjar, Microsoft Clarity, HubSpot, Segment, Adobe Analytics, and many more). Use this skill whenever the user asks to check, audit, verify, or list the tracking tags / pixels / analytics scripts on a site, asks "what tracking is on this site," wants to confirm a tag fired correctly after an implementation, is debugging duplicate or missing tags, or wants a tag inventory for a client site as part of an SEO/marketing audit. Trigger even if the user only names a specific tag (e.g. "is GA4 firing on example.com?") or only gives a domain/URL alongside audit-style language.
---

# Marketing Tag Checker

Detects marketing, analytics, and advertising tracking tags on a website's homepage by
combining a static HTML scan with a live headless-browser check. The headless-browser pass
is required to catch tags that get injected client-side (most commonly: anything fired
through a GTM container, or tags gated behind a consent banner) that never show up in the
raw HTML response.

## Workflow

**Use the headless-browser path (Claude in Chrome) as the primary method.** It's tempting to
just `web_fetch` the homepage and regex the result, but `web_fetch` returns a
content-extracted/markdown version of the page, not raw HTML — `<script>` tags and inline JS
are stripped out, so it will miss most tags entirely (it may occasionally surface a
`<noscript>` fallback pixel, but that's incidental, not reliable). Don't treat a `web_fetch`-based
scan as a real result.

### Step 1 — Load the page in a real browser

Use Claude in Chrome:
1. `navigate` to the homepage URL the user gave you.
2. **Wait for delayed scripts, don't just check immediately.** Many sites use a
   performance-optimization plugin (WP Rocket's "Delay JavaScript Execution," Perfmatters,
   Flying Scripts, etc.) that deliberately holds back marketing/analytics scripts until a
   timeout elapses or the visitor's first interaction/scroll/mouse-move — specifically so
   they don't count against page-speed scores. On a freshly-loaded page this means
   `document.scripts`, `window.dataLayer`, and every JS-global check in steps 3-4 can come
   back completely empty even though the site runs a full tracking stack. This is a real,
   observed failure mode, not a hypothetical edge case — don't report "no tags found" based
   on an immediate check alone.

   Before relying on a clean/empty result, do at least one of:
   - Wait several seconds (idle time can be enough to trigger the delay timeout), then re-run
     the checks in steps 2-4.
   - Trigger an interaction first — e.g. a small mouse move or scroll via `javascript_tool`
     (`window.scrollBy(0, 200)` or dispatching a `mousemove` event) — then re-check.
   - Re-run `read_network_requests` and the JS-globals/GTM-object checks a second time after
     the wait/interaction and compare against the first pass. If the second pass finds new
     scripts or globals that weren't there before, use the *second* pass as the real result.

   Only report a tag as genuinely absent once you've confirmed the result is stable across
   an immediate check and a post-wait/post-interaction check — not just the first one.

### Step 2 — Get the real rendered HTML and run the static scanner

Use `javascript_tool` to pull the live DOM's HTML (this captures `<script>` tags that a
markdown extractor would strip):
```js
document.documentElement.outerHTML
```
Save that output to a file, e.g. `/home/claude/homepage.html`, then run:
```bash
python3 /mnt/skills/.../ig-marketing-tag-checker/scripts/detect_tags.py /home/claude/homepage.html
```
(Adjust the path to wherever this skill is actually mounted.) This returns a JSON array of
every known tag with `found` (bool), `id` (extracted ID if any), and `matched_pattern`.
This step catches tags that are present in the page's DOM/source. It will still miss tags
that only fire after a delay, a user interaction, or consent — that's what steps 3-4 are for.

### Step 3 — Network requests (catches anything injected client-side)

Use `read_network_requests` to pull the outbound requests fired by the page. Compare request
URLs against the **network domain** column in `references/signatures.md`. This is the most
reliable signal for tags fired through a GTM container or any other dynamic loader, since
those never appear in the static HTML at all.

### Step 4 — JS globals (confirms a tag's library is loaded, even if not yet firing)

Use `javascript_tool` to check for the presence of the **JS global** column in
`references/signatures.md`, e.g.:
```js
({
  dataLayer: !!window.dataLayer,
  gtag: typeof window.gtag === 'function',
  fbq: typeof window.fbq === 'function',
  ttq: !!window.ttq,
  hj: typeof window.hj === 'function',
  clarity: typeof window.clarity === 'function',
  _hsq: !!window._hsq,
  analytics: !!window.analytics,
  uetq: !!window.uetq,
  linkedin_partner_id: window._linkedin_partner_id || null,
  // add others from the reference table as needed
})
```
If a CMP (OneTrust, Cookiebot, Osano, TrustArc) is detected, note that some tags' network
calls may only fire after consent — check `references/signatures.md` for the relevant
globals, and mention this caveat in the output if it looks relevant (e.g. a tag's script
tag is present in the HTML but its corresponding network call never appeared). Don't treat
the mere presence of a cookie/consent-related DOM element or script ID (e.g. a plugin script
tag with "cookie" in its id) as proof a real consent banner is gating anything — verify there's
an actual visible banner before citing consent-gating as the explanation for a tag not firing.

### Step 4b — Inspect the GTM container object for configured-but-not-firing tags

A tag can be fully configured inside a GTM container and never produce a network request
during a simple page load (trigger conditions like scroll depth, time delay, or a specific
page path that wasn't hit). This is a real, observed failure mode — don't skip this step
just because step 3 found nothing for a given tag.

If `window.google_tag_manager` exists, inspect its keys directly — GTM stores each
configured tag's ID as a top-level key on this object, regardless of whether it has fired:
```js
window.google_tag_manager
  ? Object.keys(window.google_tag_manager).filter(k => /^(G-|GTM-|AW-|UA-|DC-|GT-)/.test(k))
  : []
```
Any ID that shows up here but had no corresponding network request from steps 2-3 should be
reported as **"Configured, not confirmed firing"** rather than a plain ✅ or ❌ — it's a
real, distinct state: the tag exists in the container but isn't necessarily collecting data
on this page load. Don't quietly upgrade it to a full ✅ just because the ID exists somewhere.

To raise confidence one step further (optional, only if the user wants to dig in), you can
also check whether the network endpoint is reachable at all — e.g. a direct
`fetch('https://www.google-analytics.com/g/collect?...', {mode:'no-cors'})` from
`javascript_tool` — to rule out the request being blocked outright vs. simply never
triggered. A successful (non-throwing) fetch means it's not a blocking issue; the tag's
trigger just didn't fire.

### If Claude in Chrome isn't connected

Fall back to `web_fetch` on the homepage URL and run the static scanner on whatever content
comes back, but tell the user plainly that this fallback only catches tags that happen to
appear in extracted page text (mainly `<noscript>` fallback pixels) and will likely miss
GTM-injected tags, JS-only tags, and anything consent-gated. Recommend connecting Claude in
Chrome for a real result rather than presenting the fallback as comprehensive.

### Step 5 — Merge results and report

Combine the static scan, the network requests, the JS globals check, and the GTM container
introspection (step 4b) into one verdict per tag. Use **three states**, not a binary:

- ✅ **Firing** — confirmed via static HTML/script tag, an actual network request, or both.
- ⚠️ **Configured, not confirmed firing** — found only inside the GTM container object (or
  similarly, a script tag present but with zero corresponding network activity); the tag
  exists but you have no evidence it's actually collecting data on this page load.
- ❌ **Not found** — none of the methods detected it anywhere.

Pull the clearest available ID/container value (prefer one extracted from the static scan or
a network request param over a bare global check or GTM-object key).

**The final output to the person is the table — nothing else.** No preamble, no summary
paragraph, no restating what you did, no "Not found" list, no row for tags that weren't
detected. Just a table of the tags that were actually found (✅ Firing or ⚠️
Configured-not-confirmed-firing).

Fold any caveat (delayed-loading observed, consent-gating suspected, duplicate installs, a
Zaraz/server-side-proxy blind spot, an unidentified tracking-like request) into the **Notes**
column of whichever specific row it applies to — don't add a separate row or paragraph for
it. If a caveat is genuinely about the whole scan rather than one tag (e.g. delayed script
loading affecting everything), attach it to the Notes column of the most relevant row (e.g.
GTM, since that's usually the container everything else loads through) rather than
inventing a row that doesn't correspond to a real tag.

Output format:

| Tag | Status | ID / Container | Notes |
|---|---|---|---|
| Google Tag Manager | ✅ Firing | GTM-ABCD123 | Script tag + Network; site delays script loading, checked post-wait |
| Meta Pixel | ✅ Firing | 1234567890123456 | Network only (JS-injected via GTM) |
| Google Analytics (GA4) | ⚠️ Configured, not confirmed firing | G-ABC1234 | In `google_tag_manager` object; no network hit observed |

Still run the full signature list from `references/signatures.md` during detection — the
"not found" tags just don't get a row in the final output. If the user asks specifically
whether a given tag is present and it isn't, answer that directly in text; don't add it to
the table.

## Notes

- **Output format: a table of found/configured tags only — nothing else.** No intro
  sentence, no closing summary, no "here's what I found" framing, no "Not found" list, no
  row for tags that weren't detected.
- This skill checks the **homepage only** by default. If the user wants other pages checked
  too, repeat the workflow per URL — don't assume tags are identical sitewide.
- `references/signatures.md` is the source of truth for what counts as "found" and how —
  consult it whenever you need the network domain or JS global for a specific tag, and
  extend it if the user asks about a tag not yet listed (then add a matching entry + regex
  to `scripts/detect_tags.py` if it's likely to come up again).
