---
name: daily-brief
description: "Jackson's Daily Brief — Asana tasks due/overdue and email followups, sent as a Slack self-DM. Triggers: \"daily brief\", \"daily digest\", \"morning brief\", \"brief me\"."
---

Brief Jackson Price each morning like a secretary: what's on his plate, what's due soon, and what he's waiting on. One glanceable brief, delivered as a Slack DM to himself.

**Hard rules:**
- **Read-only.** Never create, edit, complete, or reschedule anything in Asana or Gmail. The single permitted write is delivering the finished brief as a Slack DM to Jackson himself — never to anyone else.
- **Prompt-free.** Run with zero interactive questions — type a trigger phrase and get the brief, no back-and-forth. If a connector fails or data is missing, note it inline and continue — never stop to ask.
- **Weekdays only.** "Yesterday" means the previous business day (Monday's yesterday = Friday). The look-ahead window counts business days and skips weekends.

## Output format

No emojis. Bold section headers, short one-line bullets. Prioritized, not exhaustive — a brief, not a feed. **Empty sections do not render.** The first line is the title only (`Daily Brief — [day], [date]`) — no subtitle, status, or caveat lines beneath it. One screen.

```
Daily Brief — Thursday, June 12

On your plate today
• Finalize Q3 launch one-pager — due today
• Send competitive battlecard to sales — due today
• Review Maya's webinar slides — from Tue (overdue 2d)
• Draft "AI in validation" blog — from Mon (overdue 3d)

Upcoming deadlines
• Competitor teardown deck — Fri
• Weekly Update section for Brian — Fri
• Customer-story interview prep — Mon

Awaiting reply (email)
• Blog draft — sent to Sarah on Tue
```

**Format rules:**
- Bullet `•`, one line each, telegraphic, no periods.
- Plate items: `task — due today`, or for carryover `task — from [day] (overdue Nd)` where N is business days past due. For carryover more than 5 business days overdue, replace the weekday with the due date (`task — from May 15 (overdue 20d)`) — a bare weekday is ambiguous that far back.
- Upcoming deadlines: `task — [day]`.
- Subtasks: if a plate or upcoming task is a subtask, append its parent for context — `task (Parent) — [day]`. Standalone tasks (even those filed under a project) get no parens.
- One screen. If a section runs long, keep the top items and drop the tail.

## Data gathering

Run both sources. **Fetch Asana and Gmail concurrently — they're independent.** If a connector is unavailable or errors, add a one-line note in the brief (e.g., `Asana unavailable — plate skipped`) and continue.

### 1. Asana — the spine

Query: tasks **assigned to Jackson**, **not complete**, **due within the 3-business-day window OR already overdue**. The window is **3 business days counting today as day 1** — Monday → Mon/Tue/Wed; Thursday → Thu/Fri/Mon.

- **On your plate today** = due today **plus** all overdue-incomplete (carryover). Carryover stays here every day until checked off, tagged `from [day] (overdue Nd)`. **Order newest-due-first:** due-today items at the top, then overdue from fewest to most days overdue (smallest `overdue Nd` first, oldest at the bottom).
- **Upcoming deadlines** = due after today but inside the window (business days 2–3).
- **Subtask context.** Render subtasks with their parent task in parens (e.g. `Outline case study, send to Dorian (Roche)`) so the bullet isn't context-free. Get the parent name **inline in the same Asana query** — request `parent` and `parent.name` via `opt_fields`; never make a separate per-task request to resolve parents (that one-call-per-task fan-out is a needless slowdown). If a task has no parent it's standalone (no parens); a task's project is not its parent — only a true parent task gets parens.

The digest is only as good as Jackson's due-date discipline; undated tasks are invisible by design. Never infer a due date.

### 2. Gmail — open loops (outbound)

Powers **"Awaiting reply (email)"** — things Jackson sent that are still hanging.

- Scan Jackson's **sent** mail from the last 1–3 business days.
- Flag a thread only when **all** hold: it reads as an ask (a question, a review request, "can you", "thoughts?", or a shared doc link); there's **no reply from the recipient**; and it's aged **past a one-business-day grace** (sent yesterday → stay silent; sent 2+ business days ago with no reply → surface it).
- Output neutral: `Blog draft — sent to Sarah on Tue`. No "no reply", no day-count, no "bump?".
- **Gmail only.** No cross-channel reply resolution — if they replied in Slack, Jackson reconciles that himself. Cross-channel chronology is deliberately out of scope.

## Delivery

Compose the brief, then send it as a **Slack DM from Jackson to himself** via the Slack connector — a real Slack message that lands in his Slack, not just printed output. That self-DM is the one permitted write.

Also print the brief in the run output, not just the DM, so Jackson can eyeball it in the chat alongside the Slack message.

## Source hygiene (assumptions this skill relies on)

One upstream habit the brief's accuracy depends on. Documented here so the dependency is explicit; the skill does not enforce it.
- **Asana due dates.** Set them on everything that's real work. Undated = invisible to the digest by design.

## Quality bar (self-check before delivering)

- Readable in under 30 seconds, one screen? If not, cut the tails.
- Any task with a guessed or inferred due date? Remove — due dates come only from Asana.
- Did any connector fail? If so, the brief says so in one line rather than dropping it silently.
- Read-only confirmed — nothing written anywhere except the self-DM?
