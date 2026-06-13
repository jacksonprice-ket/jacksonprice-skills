---
name: weekly-update-drafter
description: Draft Jackson's Product Marketing section for the Ketryx Marketing Weekly Update doc. Use this skill whenever Jackson asks to draft, prep, or pull together his weekly update, Friday update, weekly bullets, or his section of the Weekly Update doc, or when he responds to the Friday Asana ping or Brian's Friday Slack ping asking what he worked on. Also trigger on phrases like "what did I do this week", "summarize my week for the update", or "draft my PMM update". Gathers evidence from Granola meetings, Asana tasks, Slack, and Google Calendar, then produces an edit-ready draft in the exact doc format.
---

# Weekly Update Drafter

Draft Jackson Price's **Product Marketing - Jackson** section of the Marketing Weekly Update Google Doc. The output is a draft for Jackson to edit down and paste in himself. Never write directly into the Weekly Update doc and never post the update to Slack on his behalf.

## Output format

Match Jackson's exact style. Every bullet follows the pattern `[Workstream] – [telegraphic outcome fragment]`, with an en dash separator. Real example of the target output:

```
Product Marketing - Jackson

* Done this week
   * Onboarding – setup, trainings, HR, intro conversations
   * eQMS launch – caught up, introductory debrief with Carlton
   * Gartner briefing – finalized deck, Kevin sent to analysts
   * Design Controls blog – near finalized, resharing with Erez for Monday post; started accompanying deck
   * AWS Reinvent – Kevin submitted
   * Phillips case study – AE followup to James kickoff
   * Roche case study – kickoff with Rich
* Focus for next week - include the why they are your focus areas
   * Product/Marketing sync – prepare to lead biweekly starting next week, taking over from Katie
* Blockers / Support needed
* Risks
```

Format rules:
- Telegraphic fragments, not sentences. No periods. Comma-separated sub-outcomes within a bullet; semicolons to chain a second action ("near finalized, resharing with Erez for Monday post; started accompanying deck").
- One bullet per workstream. The workstream label comes first, then the en dash, then what happened.
- **Focus items embed the why naturally inside the fragment** ("taking over from Katie", "Erez posting Monday", "Brian needs before Gartner call Wed"). No "Why:" label, no separate clause structure. The why is usually a person, a date, or a dependency.
- Blockers and Risks: leave the header empty if there's nothing real. Do not write "None" and do not invent filler.
- 5-9 bullets under Done, 2-5 under Focus. Merge or cut beyond that; density beats completeness.
- After the draft, add a short **"Flagged for your review"** list (outside the update itself): items where evidence was thin, attribution was uncertain, or something seemed in-flight but unconfirmed.

## Voice

- Punchy, direct, telegraphic. Lead with the workstream, then the outcome.
- Outcome verbs and nouns: finalized, kickoff, debrief, submitted, shipped, aligned, drafted. Avoid "worked on", "continued", "attended", "met with" unless the meeting itself was the deliverable (e.g., an analyst briefing).
- En dashes (–) separate workstream from detail; that's the house format. No em dashes (—). Never use "ensure", "guarantee", "seamless", or "revolutionize".
- Name collaborators when it adds signal ("debrief with Carlton", "Kevin sent to analysts"), not as filler.

## Data gathering

Pull from these sources, in this order. Run what's available; if a connector fails or is missing, say so and continue with the rest.

### 1. Granola (highest signal)

- `list_meetings` with `time_range: this_week` (Mon through Fri of the current week).
- For the forward look, also check next week's calendar (step 4) rather than Granola.
- Pull summaries via `get_meetings` for work meetings. Skip: HR/benefits, onboarding sessions, personal entries, and intro 1:1s unless something concrete came out of them.
- Solo entries (Jackson is the only participant) are usually him working through a deliverable out loud. Treat these as evidence of work product, often high value.

**In-person speaker caveat (important):** Granola does not differentiate speakers on in-person meetings. For any meeting that may have been in person, do NOT attribute specific statements, decisions, or commitments to a named person based on the transcript alone. Instead:
- Phrase bullets around outcomes ("Aligned on June vs. September launch split") rather than attributions ("Brian decided...").
- If attribution matters for a bullet and is uncertain, put it in "Flagged for your review" with the question (e.g., "Did Brian or Gabriel own the follow-up on X?").
- Never invent a speaker.

### 2. Asana

- Tasks completed by Jackson this week → candidates for Done.
- Tasks assigned to Jackson due next week or marked in progress → candidates for Focus.
- Cross-check against Granola: a task plus a meeting on the same topic usually collapses into one stronger bullet.

### 3. Slack

- Search Jackson's messages from this week for shipped assets, approvals, feedback rounds, and blockers raised in threads (work that never hit a meeting or a task).
- Brian's Friday auto-ping thread from prior weeks can show recurring phrasing Brian responds well to.

### 4. Google Calendar

- Next week's events → anchors for Focus items and their whys (a stakeholder meeting, a launch date, a briefing).
- This week's events fill gaps where Granola has no note.

## Drafting procedure

1. Gather from all four sources.
2. Build a raw inventory of candidate items, each tagged with its evidence (meeting, task, thread, event).
3. Filter: outcomes only. A week with 35 meetings must not read like a calendar dump.
4. Merge by workstream. Typical Jackson workstreams: eQMS GTM, thought leadership / Erez content, analyst relations (Gartner), competitive (battlecards), sales enablement skills, buyer journey / content gap work. Use whatever the evidence shows; don't force these categories.
5. Write Done, then Focus (with whys), then honestly assess Blockers and Risks. If nothing surfaced, write "None" rather than inventing filler.
6. Append "Flagged for your review".
7. Deliver as a chat draft (or a markdown block Jackson can copy). Do not write to the Google Doc.

## Quality bar (self-check before delivering)

- Would Brian learn something from each bullet, or is it activity noise?
- Does every Focus item have a why?
- Any bullet attributing words or decisions to a person based on an in-person transcript? Remove or flag it.
- Any banned words or em dashes? Remove.
- Is the whole section readable in under 60 seconds?
