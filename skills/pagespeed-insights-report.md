---
name: "pagespeed-insights-report"
description: "Run Google PageSpeed Insights for a URL using the built-in browser and return mobile and desktop scores, Core Web Vitals, and top fixes in a report-ready table. Use for page speed, Lighthouse, or Core Web Vitals checks."
---

# PageSpeed Insights Report

Runs pagespeed.web.dev in the built-in browser (the browser pane in the Claude desktop app) for one or more URLs and returns a clean mobile and desktop summary the strategist can paste into a client report. This skill does not use the PageSpeed API or any API key. The API is not reachable from the cloud session, so everything goes through the built-in browser.

Read the built-in-browser skill first if it has not been read in this session. Its tools are named mcp__remote-devices__Claude_Browser__* (load them all in one ToolSearch call using that prefix).

## Inputs

- One or more full URLs (include https://). If the user gives only a domain, use the homepage as https://domain.
- If the user gives a client name with no URL, ask for the URL once. Do not guess.
- Default to the homepage. Test additional URLs only when the user lists them.
- Run one URL at a time, in order.

## Step 1: Get access and open the report

1. Call request_access for https://pagespeed.web.dev with scope "site" (so later runs do not ask again), then retry the page open.
2. Open this address with preview_start (it returns a tabId):
   `https://pagespeed.web.dev/analysis?url=<URL-ENCODED URL>&form_factor=mobile`
3. The page redirects to a report address and runs a live test. It takes 30 to 90 seconds. Wait in 10 second steps. The result is ready when the page shows score numbers (Performance, Accessibility, Best Practices, SEO). While running, the text only shows a spinner and the words "Diagnose performance issues".
4. If the page has not produced scores after about 3 minutes, tell the user the test did not finish and ask whether to retry. Do not loop more than once.

## Step 2: Read the mobile results

Use get_page_text on the tab. For the mobile report this returns clean text starting with the four scores, then the metrics (First Contentful Paint, Largest Contentful Paint, Total Blocking Time, Cumulative Layout Shift, Speed Index), then an "Insights" list and a "Diagnostics" list with estimated savings, for example "Render-blocking requests Est savings of 3,630 ms" or "Improve image delivery Est savings of 273 KiB".

Field data (real users) is the section "Discover what your real users are experiencing". If it says "No Data" or "does not have sufficient real-world speed data", report "No field data available (not enough real-user traffic)" and rely on lab data. If field data is present, record LCP, INP, CLS and FCP with their Good, Needs Improvement or Poor labels.

Record the report date shown at the top ("Report from ...").

## Step 3: Switch to desktop and read the results

1. Use find with the query "Desktop" and click that element. The address changes to form_factor=desktop. The desktop test runs on its own, so wait in 10 second steps until a screenshot (scale 0.5) shows desktop score circles.
2. Do not use get_page_text for desktop. On the desktop tab it returns a wall of stylesheet code. The page holds both reports in the page at once. Use the javascript_tool instead to read only the visible report with this code:

```
(()=>{const clean=n=>{const c=n.cloneNode(true);c.querySelectorAll('style,script,svg,noscript,table,.lh-details').forEach(e=>e.remove());return c.textContent.replace(/\s+/g,' ')};const roots=[...document.querySelectorAll('.lh-root')];const vis=roots.filter(r=>r.offsetParent!==null);const r=(vis[0]||roots[roots.length-1]);const t=clean(r);const i=t.indexOf('Values are estimated');return t.slice(Math.max(0,i),i+4500)})()
```

3. Take the four scores from a screenshot of the desktop tab (the circles under "Diagnose performance issues"), because the script text starts at the metrics.
4. Check that the metric values differ from the mobile ones and that the text says "Emulated Desktop". If it still says "Emulated Moto G Power", the desktop report has not loaded yet. Wait and read again.

## Step 4: Check for warnings

Lighthouse sometimes prints "There were issues affecting this run of Lighthouse" (for example "The page loaded too slowly to finish within the time limit. Results may be incomplete."). If that appears on either report, say so in the output and note that the affected numbers may be unreliable. Offer one re-run with the Analyze button. Do not re-run more than once without asking.

## Step 5: Score bands

Google's bands: 90 to 100 Good, 50 to 89 Needs Improvement, 0 to 49 Poor.
Core Web Vitals thresholds (Good / Needs Improvement / Poor):
- LCP: 2.5 s or less / up to 4.0 s / over 4.0 s
- INP: 200 ms or less / up to 500 ms / over 500 ms (field data only, lab runs do not report INP)
- CLS: 0.1 or less / up to 0.25 / over 0.25
- FCP: 1.8 s or less / up to 3.0 s / over 3.0 s
- TBT: 200 ms or less / up to 600 ms / over 600 ms
- Speed Index: 3.4 s or less / up to 5.8 s / over 5.8 s

## Step 6: Output format

For each URL, return:

1. A scores table with rows Performance, Accessibility, Best Practices, SEO and columns Mobile, Desktop, each cell showing the number and band. Leave out the "Agentic Browsing" score unless the user asks for it.
2. A Core Web Vitals table with rows LCP, INP, CLS, FCP, TBT, Speed Index and columns Mobile, Desktop. Mark which values are field data and which are lab data. INP shows "n/a (field data only)" when there is no field data.
3. The top 5 fixes for mobile and the top 5 for desktop, taken from the Insights and Diagnostics lists, ranked by estimated time savings in ms first, then by KiB. Write each in plain language a client could follow (for example, "Defer or inline CSS that blocks the page from showing, saves about 3.6 s"). Name the specific cause when the page lists it (for example, many separate Elementor stylesheet files).
4. One short plain-language takeaway (two or three sentences) that names the biggest issue and the single most valuable fix.
5. A footnote with the test date and a reminder that lab scores vary run to run, so a change of a few points is noise.

When testing several URLs, add one summary table at the top with each URL's mobile and desktop performance score.

## Rules

- Mobile is the headline number. Lead with it.
- Never invent or estimate scores. If a value could not be read, write "n/a" and say why.
- If the user declines site access, tell them the report cannot run without it and stop.
- Do not use em dashes in any output.
- Do not add sections, recommendations, or metrics beyond what is listed here unless the user asks.
- Flag inconsistencies without being asked (for example, a large gap between mobile and desktop scores, a Lighthouse warning, or a redirect where the tested URL differs from the requested URL).
- If the user asks for a comparison to a previous run, ask for the earlier numbers rather than assuming them.
- If the built-in browser cannot be reached (the desktop app is closed or offline), say so and offer to format results the user pastes from pagespeed.web.dev.
- Offer to write the results into a client report only after delivering the tables.