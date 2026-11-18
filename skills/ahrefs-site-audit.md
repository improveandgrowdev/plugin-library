---
name: ahrefs-site-audit
description: >
  Use this skill whenever the user wants to retrieve, view, or analyze Ahrefs Site Audit data
  for a website. Triggers include any mention of "site audit", "audit data", "SEO health",
  "crawl issues", "site health score", "Ahrefs audit", or requests like "run a site audit",
  "show me the audit for [site]", "what issues does [site] have", "check my site health",
  or "get the latest crawl data". Always use this skill when the user mentions a domain or
  URL alongside any audit, crawl, issue, or health-related language — even if they don't
  say "Ahrefs" explicitly.
---

# Ahrefs Site Audit Skill

Retrieve and present the latest Ahrefs Site Audit data for a user-specified site, enrich the report with the matching ClickUp task, then (after user confirmation) write the report back to that task's description.

---

## Step 1: Identify the site

Extract the target site from the user's message. If no site is specified, ask for it before proceeding.

Derive a **short site name** (e.g. `example.com` → `example`, or use the brand name if obvious). You'll use this for fuzzy-matching against ClickUp folder names.

---

## Step 2: Find the matching Ahrefs project

Call `Ahrefs:site-audit-projects` with no arguments to list all projects:

```
Ahrefs:site-audit-projects()
```

Match the user's site to a project by comparing the `project_url` field (case-insensitive, strip `www.` and trailing slashes). If no match is found, tell the user the site doesn't have an Ahrefs Site Audit project set up, and stop.

---

## Step 3: Fetch the latest audit summary

Once you have the `project_id`, call:

```
Ahrefs:site-audit-projects(project_id=<id>)
```

This returns the latest crawl's health score, issue counts, and crawled page counts.

---

## Step 4: Fetch all issues

Call:

```
Ahrefs:site-audit-issues(project_id=<id>)
```

This returns all issues from the latest crawl.

---

## Step 5: Find the matching ClickUp folder and task

**Run Steps 5a and 5b sequentially** — you need the folder ID before you can filter tasks.

### 5a — Find the matching ClickUp folder

Call `ClickUp:clickup_get_workspace_hierarchy` with `max_depth=1` to get all spaces and folders:

```
ClickUp:clickup_get_workspace_hierarchy(max_depth=1)
```

Find the folder whose name best matches the client's domain or site name (case-insensitive, partial/fuzzy match — e.g. folder named "Acme" matches domain `acme.com`; strip `www.`, `.com`, `.org` etc. when comparing). Note the `folder_id`.

If no folder matches, skip the ClickUp section and note in the report that no matching ClickUp folder was found.

### 5b — Find the "Technical SEO Audit" task

Once you have the `folder_id`, retrieve tasks using the current month's date range as a filter:

```
ClickUp:clickup_filter_tasks(
  folder_ids=[<folder_id>],
  include_closed=false,
  due_date_from=<first day of current month, YYYY-MM-DD>,
  due_date_to=<last day of current month, YYYY-MM-DD>,
  statuses=["to do"],
  order_by="due_date",
  reverse=false
)
```

From the results, filter for tasks that meet **all three** of these criteria:
1. Task name contains "Technical SEO Audit" (case-insensitive)
2. Status is **"to do"** — exclude any task that is closed or has any other status
3. Due date falls within the **current month and year** (enforced by the date filter above)

Pick the task that matches. If multiple match, prefer the one with the earliest due date. Note the `task_id`, status, assignees, due date, and task URL.

If no matching task is found, note in the report that no "Technical SEO Audit" task with a "to do" status due this month was found in the matching ClickUp folder.

---

## Step 6: Present the full report

Output a single Markdown document with the following sections:

---

### Site Audit Report: [domain] — [crawl date]

**Health Score:** [X / 100]  
**Pages Crawled:** [N]  
**Crawl Date:** [date]

---

### Active Issues

List **all active issues** in a Markdown table with these columns:

| Severity | Issue | Affected URLs | Description |
|----------|-------|---------------|-------------|

Only include **Errors** and **Warnings** — exclude Notices entirely.

Sort by: Errors first, then Warnings. Within each group, sort by affected URL count descending.

Only include issues where affected URL count > 0.

---

### ⚡ Quick Win (15–20 minutes)

Identify up to **two** quick win options from the active issues. Each option must meet all of these criteria:
- Fix is simple and repeatable (e.g. adding a missing tag, fixing a redirect, updating a meta description)
- Affects a meaningful number of pages
- Does not require developer involvement or significant content work

If fewer than two qualifying issues exist, only list what's there — it's fine to have just one option or none.

For each option, write 2–3 sentences explaining:
- What the issue is
- Why it matters for SEO
- Exactly what to do to fix it (actionable steps)

Label them **Option 1** and **Option 2** if two are present.

---

### 📋 1-Hour Project (for Account Strategist)

Pick a higher-impact issue or cluster of related issues that:
- Has significant SEO value if resolved
- Can be scoped into a clear, presentable recommendation
- Is achievable within roughly one hour of focused work

Write this section as a short brief suitable for presenting to an account strategist:
- **Issue:** What's wrong
- **Impact:** Why this matters (traffic, rankings, crawlability, etc.)
- **Recommendation:** What to do, step by step
- **Expected outcome:** What improvement to expect

---

### 📌 ClickUp Task

> Include this section only if a matching task was found in Step 5. If no task was found, add a brief note (e.g. "No 'Technical SEO Audit' task found in the [Folder Name] folder.") and skip Step 7.

| Field | Value |
|-------|-------|
| **Task** | [Task Name] |
| **Status** | [status] |
| **Assignee(s)** | [names, comma-separated] |
| **Due Date** | [due date or "None"] |
| **Link** | [ClickUp task URL] |

---

## Step 7: Ask for confirmation before writing to ClickUp

After presenting the full report, ask the user:

> "Would you like me to update the ClickUp task **[Task Name]** with this report? This will **replace** the existing task description entirely."

**Do not call `ClickUp:clickup_update_task` until the user explicitly confirms.**

---

## Step 8: Write the report to ClickUp (on confirmation only)

Once the user confirms, call:

```
ClickUp:clickup_update_task(
  task_id=<task_id>,
  markdown_description=<full report markdown>
)
```

Use the complete Markdown report from Step 6 as the `markdown_description`. After a successful update, confirm to the user with a direct link to the task.

---

## Notes

- Always use the **latest** crawl (no `date` parameter needed — the API defaults to the most recent).
- If the user asks for historical data or a specific crawl date, pass the `date` parameter (format: `YYYY-MM-DDThh:mm:ssZ`).
- If the user wants page-level detail for a specific issue, use `Ahrefs:site-audit-page-explorer` with the relevant `issue_id`.
- The Quick Win and 1-Hour Project recommendations should be based on the actual issues present — use your SEO knowledge to reason about effort, impact, and what an account strategist would find compelling. Do not make up issues that aren't in the data.
- Folder name matching should be fuzzy — strip `www.`, `.com`, `.org` etc. and compare the core name against folder names case-insensitively.
