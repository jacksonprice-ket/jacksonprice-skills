---
name: call-to-tasks
description: >-
  Turn a specific Granola call into reviewed Asana tasks for Jackson — pull a call's action items / next steps into his task list. Triggers: conversational forms ("process my last call", "analyze my last call", "call to tasks", "turn my call into tasks") and a terse form naming a meeting with an optional day — keyword ("asana" or "tasks") + meeting reference + optional time qualifier, e.g. "asana recent roche sync", "tasks yesterday's brian 1:1", "asana today's brian jackson call", "tasks yesterday's bailey meeting". The trigger always names a specific call; a bare Asana request with no meeting ("what are my asana tasks", "add a task", "tasks due this week") is NOT this skill. Creates Jackson's OWN tasks from a call — not sales-asset recs for a prospect (asset-recommender), not his weekly update (weekly-update-drafter).
---

# Call-to-Tasks

Turn Jackson Price's most recent Granola call into reviewed Asana tasks. The loop: read the meeting, propose tasks in chat, write to Asana only after Jackson approves. The human approval gate is the whole point — nothing reaches Asana unreviewed.

## Hard rules

- **Approval gate, always.** Propose tasks in chat first; write to Asana ONLY after Jackson explicitly approves ("yes", "yes but…"). No silent writes, ever — even if asked to "just add them." This is the responsible-AI spine of the skill.
- **Approval is editable.** "Yes but drop #3", "change due on #2 to Friday", "promote maybe #1" are all valid. Apply the edits, then write only what survived the review.
- **File into My Tasks sections by due date.** Create tasks assigned to Jackson (no project/board), and route each into the correct My Tasks section by its due date — see "Section routing" below.
- **Read Granola freely; write nothing else.** The only permitted write is creating approved Asana tasks, including their My Tasks section placement. Never edit, complete, or delete existing tasks; never create, rename, or reorder sections; never touch automation rules; never post to Slack. The connector files tasks into sections that already exist — it cannot create one — so the three target sections must already exist in Jackson's My Tasks ("Due today", "Upcoming", "Later or unscheduled").
- **Never invent.** Absence of a date means a blank date. Ambiguous ownership means the maybes bucket. Never guess a due date or a speaker.

## Which meeting

The trigger usually names the meeting. Resolve it from its two parts:

- **Time qualifier** ("today's", "yesterday's", "this morning's", "recent", "last"): sets the `list_meetings` window — `this_week` covers today/yesterday; widen to `last_30_days` for "recent" / "last" if nothing matches. "Yesterday" means the previous business day (Monday's yesterday = Friday). Resolve relative days against Jackson's local date.
- **Meeting reference** ("roche sync", "brian 1:1", "bailey meeting", "brian jackson call", "eQMS sync"): match against meeting titles and participants — "brian 1:1" → a Jackson + Brian meeting, "bailey meeting" → a meeting with Bailey, etc.

If the trigger names no meeting at all ("process my last call"), default to the most recent work meeting.

**On ambiguity, ask — don't guess.** If more than one meeting matches (two Brian meetings yesterday, "roche sync" hitting several), list the candidates with dates and let Jackson pick. One quick question beats pulling tasks from the wrong call.

## Extraction logic

Read two things from one `get_meetings` call — no transcript pass by default.

1. `get_meetings` on the target meeting returns both Granola's **AI summary / action items / next steps** and Jackson's **raw typed notes** (what he wrote live, separate from the AI text). Use both:
   - **Raw notes** are the highest-confidence signal — Jackson wrote them, so anything there is reliably his and reliably what he chose to flag.
   - **AI summary + next steps** are the backbone; they catch what Jackson didn't type.
2. Pull due-date cues from the summary and notes ("due tomorrow", "before the Gartner call", "live Monday").
3. Sort every candidate into one of two buckets:

**Proposed (confident)** — clearly Jackson's own commitment: it's in his raw notes, or the summary attributes it to Jackson on a call where attribution is trustworthy (e.g. a clean 1:1).

**Maybes (flagged)** — surface but don't assume. A candidate lands here when any of these hold:
- **Ownership unclear.** Could be Jackson's, could be a teammate's.
- **Shared-room / multi-person call.** On in-person (Owl) or multi-person calls, Granola's name attribution in the summary can be wrong. Trust Jackson's raw notes here; treat summary items attributed to him as maybes unless his notes corroborate them. When in doubt, flag; never invent who owns something.
- **Borderline.** Reads more like discussion than a real action ("we should think about X"), or a standing process habit rather than a one-off task.

Better to flag a maybe than to silently drop a real commitment or silently assign someone else's.

**Transcript fallback (opt-in).** Default is summary + notes only — it's faster, and on messy calls more reliable than the transcript (in-room speakers merge and lines scramble). Pull `get_meeting_transcript` only when Jackson asks for a deeper pass, or when his notes are thin/absent on a high-stakes clean 1:1 where a missed commitment would cost him. Say so in the proposal when you've done this, so he knows the run went deeper than usual.

### Due dates

Propose a due date ONLY when the call actually gave one — explicit ("by Thursday") or clearly implied against a known date ("before the Roche kickoff"). Otherwise leave it blank. Never infer a deadline from nothing.

## Section routing (due date → My Tasks section)

Every created task is filed into a My Tasks section based on its due date, computed against **today in Jackson's local time** (the day the skill runs):

- **Due today** → `Due today` section
- **Due tomorrow, in 2 days, or in 3 days** → `Upcoming` section
- **Due more than 3 days out** → `Later or unscheduled` section
- **No due date** → `Later or unscheduled` section
- A date already in the past counts as **Due today** — it needs attention now.

### Resolving section GIDs

`asana_create_task` places a task in a My Tasks section via its `assignee_section` parameter, which takes the section's GID. Resolve names to GIDs like this:

1. Start from the known map:
   - `Due today` → `1215695292547456`
   - `Upcoming` → `1213549888251388`
   - `Later or unscheduled` → `1213549888251389`
2. For any name not in the map, resolve at runtime: `asana_get_tasks` (assignee `me`, `opt_fields=assignee_section.name,assignee_section.gid`), then match the section by exact name and cache the GID for the rest of the run.

**Empty-section caveat.** The connector can only see a section's GID once at least one task lives in it — there is no "list all My Tasks sections" call. All three section GIDs are now captured in the map above, so routing works on a cold run. The runtime scan is only a fallback if a section is ever deleted and recreated with a new GID; in that case, if a target section's GID can't be resolved, create the task **without** `assignee_section` (it lands in the My Tasks default area) and say so in the confirmation.

## Output format

Present in chat — nothing written yet. Two buckets, numbered so Jackson can reference each item.

```
From [Meeting name] — [date]

Proposed tasks
1. Send Rich the Philips case-study outline — due Thu Jun 18 → Upcoming
2. Draft eQMS June-vs-September launch summary for Confluence — no date → Later or unscheduled
3. Share Greenlight battlecard with Bailey — due today → Due today

Maybes (unclear owner / in-person — your call)
4. Follow up with Gabriel on methodology graphic — flagged: room recording, unclear if yours
5. Book Gartner DCCA prep slot — flagged: discussed, not a clear commitment

Reply "yes" to add the Proposed tasks, or edit: "yes + 4", "drop 2", "due Fri on 1".
```

Rules: action-first task names, telegraphic, no periods. Each proposed task shows its target section (`→ Section`) derived from its due date, so Jackson sees where it will land before approving; if he re-dates a task during approval, recompute its section. Each maybe carries a short `flagged: why` so Jackson knows why it isn't confident. If a proposed task looks like it duplicates an open task Jackson already has, note it inline (`— possible dup of existing task`) rather than creating silently; a quick `asana_get_tasks` on his incomplete tasks covers this.

## Approval & write

On approval, for each surviving task call `asana_create_task`:
- `assignee`: Jackson (`me`)
- `name`: the task name as approved/edited
- `due_on`: only if a date was proposed and survived
- `assignee_section`: the GID of the My Tasks section the task routes to by due date (see Section routing). Omit only if that section's GID can't be resolved yet (empty-section caveat).
- `notes`: the source line — meeting name, date, a link if `get_meetings` returned one, and the source it came from (the summary line or Jackson's own note) — so every task is traceable back to the call.

Resolve Jackson's Asana identity once via `asana_get_user` ("me"). After writing, confirm what landed grouped by section — e.g., "Due today (1), Upcoming (2), Later or unscheduled (1)" with the task names — and call out any task that couldn't be routed to its section.

## Quality bar (self-check)

- Did I stop and show the buckets before touching Asana? If not, stop now.
- Any task carrying a due date the call didn't actually give? Blank it.
- Does each task's section match its due date (today → Due today, 1–3 days → Upcoming, 4+ days or none → Later or unscheduled)?
- On a shared-room or multi-person call, any item assigned to Jackson by the summary alone (not backed by his own notes)? Move it to maybes.
- Does every maybe carry a one-line `flagged: why`?
- After writing: only approved items created, all assigned to Jackson, nothing else in Asana touched?
