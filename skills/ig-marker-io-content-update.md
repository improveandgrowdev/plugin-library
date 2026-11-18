---
name: ig-marker-io-content-update
description: Check Marker.io for open issues that request content edits (page/post title, copy, or wording changes), but only for client sites with a live MCP connector (WordPress or other CMS) this session — skip sites you can't act on rather than scanning them anyway. Apply the fix — edit in place if the connector supports updates, else create new content with the requested change and flag it as a stand-in — then return a link to confirm. Use whenever the user asks to "check Marker.io," "check for new site feedback," "sync content edits," "action the Marker.io tickets," "process client feedback," or wants feedback/bug tickets turned into site changes, even without naming Marker.io explicitly.
---

# Marker.io Content Sync

## What this does

Marker.io collects visual feedback and bug reports on client websites. Many of
those tickets are really just content edits in disguise — "change this
heading," "update this paragraph," "swap this photo's caption" — filed by a
client or teammate against a live page. This skill closes the loop: it finds
those tickets, makes the edit on the actual site, and hands back a link so a
human can eyeball the result before anything goes live.

It intentionally does **not** try to handle every Marker.io ticket. Design
bugs, broken layouts, JS errors, and "this button doesn't look right" issues
need a developer, not a content edit — leave those alone and mention them in
your summary rather than guessing at a fix.

## Step 1 — Find out which sites we can actually act on

Before touching Marker.io at all, find out which CMS/WordPress MCP connectors
are actually available in this session — that's the hard constraint on
everything downstream, so check it first rather than last. WordPress
connectors in this environment describe themselves as "a WordPress MCP Server
... for the WordPress site NAME (https://DOMAIN)"; use `ToolSearch` (e.g.
query "wordpress" or "wp_") to enumerate them, and note each connector's
DOMAIN. If other CMS-flavored connectors exist, note their domains too.

If there are zero such connectors this session, stop here and tell the user
that — there's nothing to check, and there's no point burning calls scanning
Marker.io for sites you can't do anything about anyway.

## Step 2 — Find candidate issues, but only for those sites

List Marker.io projects with `projects_list`, excluding archived ones (leave
`archived` unset/false — an archived project is done, its site work is over,
and its tickets aren't live work regardless of what status they're in). Keep
only the non-archived projects whose `websiteUrls` (from `project_get`) match
a domain from Step 1. Everything else — the other client sites you don't have
a live connector for this session — is out of scope; don't spend time
pulling their issues or reporting on them. This is the main reason to do
Step 1 first: filtering projects down to a handful before pulling issues is
far cheaper than pulling issues for every project and discarding most of
them afterward.

For each matched project, pull issues with `issues_list` filtered to
**`status: "Open"` only** — not In Progress, not the waiting-for-approval
status, not Resolved, and not Closed. Only a strictly Open ticket represents
unactioned work; anything else has already been picked up, finished, or
otherwise moved along, and re-surfacing it just adds noise.

For a fresh "check for new issues" request, prioritize issues created since
the last time this was run, or since a timeframe the user gives you — but
always show *all* currently-open content-edit issues too, since a stale
unactioned one is just as relevant as a brand-new one. That said, if a chunk
of open issues clearly hasn't been triaged in a long time (many months or
years old, no comments, no assignee, no activity), call that out as its own
"stale backlog" group in your summary rather than silently folding it in with
fresh tickets or silently dropping it — let the user decide whether it's still
live work or dead weight. Don't make that judgment call invisibly.

Read each issue's title and description (`issue_get`) and use judgment, not a
keyword list, to decide whether it's a content-edit request. The signal is
whether the fix is "change what the words/title/image say," not "change how
the page behaves or is laid out." When in doubt, look at the screenshot or
context (`issue_get_screenshot`, `issue_get_context`) — a ticket that's really
about broken styling can *sound* like a content complaint in the title.

Set aside anything that isn't a content edit. You'll still want to mention
those to the user (e.g. "3 more open tickets, but they're layout bugs — not
something I can safely touch") so nothing silently falls through the cracks.

## Step 3 — Check whether the connector can actually edit

Don't assume. Look at the *actual* tool list for the matched connector right
now — plugins get added or removed, and a memory of "wordpress-mcp only has
wp_add_post" from last week may no longer be true. Look for an update/edit
verb (e.g. `wp_update_post`, `wp_edit_page`) alongside the create one (e.g.
`wp_add_post`). Some connectors will have both, some only creation.

This determines which path you're on:

- **Update tool exists** → fetch the current content by ID (parse the ID out
  of the issue's URL — e.g. `?p=16189` — or search for it by title/slug via
  the connector's search tool if the URL doesn't carry one), apply exactly
  the change the ticket asked for, and call the update tool.
- **No update tool, only create** → build new content with the requested
  title/copy, save it in the least disruptive state available (draft, if the
  connector supports one) rather than publishing over the original, and make
  it unmistakable in your summary that this is a *new* item standing in for
  the edit, not the original page changed in place. The old content is
  untouched — say so.

This path applies purely because of what the *tool* can do, regardless of
whether the current target happens to already be a draft itself (e.g. someone
filed a ticket against an unfinished draft, asking for different placeholder
content). Resist the urge to treat "it's just a draft anyway" as license to
skip the fallback framing — a user coming back later still needs to know
there are now two drafts, which one is the original, and which one has the
requested content, or they'll waste time reconciling them.

## Step 4 — Confirm before writing

Before calling any create/update tool, show the user what you're about to do:
site, ticket, old → new title/content (or a summary of the diff if it's long),
and which path (in-place edit vs. new-item fallback) you're taking for each
one. Batch this into one summary across all candidate issues rather than
stopping for each — the point of automating this is speed — but don't skip
the checkpoint entirely. These are real edits on real client sites; a quick
"here's what I'm about to do, proceed?" costs one round-trip and saves a
much worse one.

If a ticket's requested change is genuinely ambiguous (e.g. it references a
Google Doc or asset you don't have), don't guess — flag it back to the user
instead of half-implementing it.

## Step 5 — Apply and report back

After the user gives the go-ahead, make the calls, then report, per site:

- What changed (or what was created)
- A working link to the result
- Which path was used (edited in place vs. created new) and why

Do **not** automatically comment on or change the status of the Marker.io
issue itself afterward — that's visible to clients and teammates on the
other end. Offer to do it ("want me to comment on the ticket with this link,
or mark it resolved?") and only act once the user says yes.

## Notes from building this

- The Marker.io MCP tools and the WordPress MCP tools are separate connectors
  per site — there's no single tool that spans both, so the matching in Step
  2 is doing real work, not a formality.
- Content-edit tickets are usually filed with a URL that has the target post
  or page ID right in it as a query param (`?p=1234`) or a clean slug — check
  there before falling back to a title search.
- Draft status is the safe default for the create-fallback path: it gives the
  user something to review at the returned link without it going live on its
  own.
- Most Marker.io projects won't have a matching CMS connector available in
  any given session — that's expected, not a bug, and Step 1 exists so you
  filter those out up front instead of discovering it late. If the user
  specifically asks about a site you skipped this way, say plainly that no
  connector was available rather than pretending it was checked.
