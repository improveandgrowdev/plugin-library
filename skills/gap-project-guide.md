---
name: gap-project-guide
description: 'Guides an entire Improve & Grow (I&G) Growth Action Plan (GAP) project from intake to handoff, working alongside the user. Keeps the project state (client, revenue streams, shared facts, what is done, ready, blocked or stale), calls the right gap skill at the right time, passes along what is already known so the user is never asked twice, flags access and inputs to request early, reroutes when the user changes scope, and decides with the user whether a requested change is a one-time adaptation for this client or a real improvement to a skill. Use whenever the user starts or resumes a GAP, growth action plan, proposal deck or client plan, asks what is next, what is left, whether anything is stale or out of date after a change, whether to update a skill, or says things like "let''s work on the Pilgrim Pines GAP", "where are we", or "pick this back up". Also use when a gap skill''s output needs a project-level decision.'
---

# GAP: Project Guide

The coach for the whole Growth Action Plan. Each `gap-*` skill does one piece well. This skill knows how the pieces fit, what has been done, what a change breaks, and when a change belongs in a skill instead of just this project. It never redoes a skill's work: it chooses the skill, hands it the facts it needs, records the result, and keeps the project moving.

Copy rules from `improve-and-grow-brand` apply to anything written for a client. Keep every message to the user short, plain and direct. One question at a time, and only when a wrong guess would be costly.

## State lives in two places
- `gap-project.json`, kept by `scripts/project_state.py`: client, shared facts (streams, ads type, dates, competitors, brand terms), the status of every task, access requests, adaptations and decisions. Update it as work happens, not at the end.
- A tracker the user can see, written by `project_state.py report`: a markdown table with the full state embedded. Sessions end, so on the first save offer to publish it as a Claude Doc and keep it in sync; on resume, read that doc back with `restore`. Without a saved tracker a new session starts from nothing.

## Starting or resuming
1. **Resume** if a tracker or `gap-project.json` exists (ask for the doc link or look in the session). Run `project_state.py status`.
2. **Start** if not. Get from the user and ClickUp (read the Client Overview first, ask for the link only if you cannot find it): client name, the revenue streams in presentation order, whether the client has a Google Ads account (`grant`, `paid`, `none`), the presentation date and who is presenting (no default: the team builds plans under different names). That is the whole intake. Then `project_state.py init`. Do not ask for anything a later skill can find itself.
3. **Brief the user** in five lines or fewer: where the project stands, what is next, what is waiting on something, anything stale, and the access to request now. Then start the next task unless the user redirects. See `references/session-routine.md`.

## Running the project
- **Pick the work with `project_state.py next`.** It respects dependencies (`references/skill-map.md`), puts stale redos first, and lists access to line up. Tell the user which skill you are about to use and why in one line, then invoke it with the Skill tool.
- **Hand over known facts.** Before invoking a skill, state the shared facts it needs (streams, competitors, period, brand terms, location, targets) so it does not ask again. After it finishes, record what it produced: `project_state.py set TASK done --artifact FILE`, and `ctx` for new shared facts (for example the competitor list from share of voice).
- **Line up access early.** GA4, Google Ads, Ahrefs projects, the client's revenue numbers and the HighLevel or BrightLocal report take time. Name them in the first brief, track them with `project_state.py access`, and offer to draft the request to the client. Do not send anything for the user without a yes.
- **Gate the handoffs between phases.** Before keyword research, confirm the stream list is final. Before the budget model, confirm goals for every stream and the services in scope. Before assembly, run `project_state.py status` and settle every stale or open task. After assembly always run `gap-deck-review`, and again after any later change to a slide.
- **Watch the date.** If the presentation is close, say what can be cut or run in parallel (for example Rank Tracker needs about a day to fill, so start keyword research for every stream first). Cut scope only with the user's approval.
- **Optional pieces** (brand messaging guide and brandscript slides) are skipped until the user wants them. Ask once when the plan phase starts.

## When the user changes something
Decide which of three things happened, then act:
1. **A fact changed** (a stream added, a revenue figure corrected, the date moved). Record it with `project_state.py ctx` or by re-running the task. Finished tasks that used it turn stale. Tell the user exactly which slides need a redo and offer to redo them in order. Never leave stale work in the deck.
2. **A one-time adaptation** (this client needs something the skill does not do by default). Apply it through the skill's own config, spec or overrides, not by editing the skill. Log it with `project_state.py adapt`.
3. **A skill should change** (the skill was wrong, or the change is for every client). Follow `references/adaptation-guide.md`: run `project_state.py adapt` with honest flags, give the user the recommendation in one or two sentences, and let them decide. If the answer is "update the skill", finish the current deliverable first using the one-time route, then update the skill afterward with `skill-creator` (copy the synced skill, edit, repackage, deliver the .skill file), clear the backlog entry, and say what changed in one line. A standard that belongs to every I&G output (voice, deck look) is changed in `improve-and-grow-brand` or `ig-deck-style`, not in each gap skill.

The recommendation is a starting point. The user decides, and a standing preference they state ("always do X") is a skill change even on the first occurrence.

## Quality gates
- Numbers match across slides: stream names and goals are identical wherever they appear. `gap-deck-review` is the final check, but catch an obvious mismatch the moment you see it.
- Nothing is called done without a file or a reason: `set done` needs the artifact path unless the skill produced no file.
- A skipped task needs a reason in its note, so the assembly report and the tracker explain the gap.
- Untested routes stay labeled untested (the browser captures for audits, rank tracker setup). Say so when a skill reports a capture failure and offer the upload backup.

## Working as a team
These skills may be used by several people on the same account, so nothing personal is baked in.
- **Shared backlog.** `gap-skill-backlog.json` is local to a session. Keep the real backlog in one shared Claude Doc ("GAP skill backlog"), read it at the start of a project and append candidates with the client name, so a recurrence seen by two people counts. Seed the local file from the doc, and write changes back.
- **Skill ownership.** A skill update is delivered as a .skill file for whoever manages the account's skills to upload. Say that plainly, name what changed, and keep the old version until it is replaced. Until the updated skill is uploaded, other people still run the old one, so note it in the tracker.
- **Access is per person.** Browser captures use the signed-in user's own Google Ads, GA4 and Ahrefs. The person running the audit needs that access, so record who holds it in the access notes.
- **Client data.** Trackers and reports hold client numbers. Publish them as private docs and share deliberately.
- **Connectors.** ClickUp, Ahrefs and any Windsor connection must be enabled for the account. If one is missing, say which, and use the skill's upload or paste backup.

## Closing a project
When the deck has passed review: update the tracker, list what was adapted for this client, show the backlog (`project_state.py backlog`) and ask which candidates to turn into skill updates now. Offer to post a short status to the client's ClickUp task. Do not do either without a yes.

## Files
- `scripts/project_state.py`: init, status, next, set, ctx, access, adapt, backlog, report, restore (run with `-h`).
- `references/registry.json` (tasks, dependencies, shared facts, access needs), `skill-map.md` (the same, readable), `session-routine.md`, `adaptation-guide.md`.
- `evals/evals.json`.
