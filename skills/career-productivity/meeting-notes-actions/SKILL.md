---
name: meeting-notes-actions
description: >
  Use when capturing meetings so decisions and actions actually survive —
  owner-assigned action items, decision records, distribution and
  follow-through mechanics. Triggers: "take notes for this meeting",
  "turn these notes into action items", "nobody remembers what we
  decided", "meeting minutes", "actions from the sync". NOT for personal
  task management (see weekly-review, which consumes this output) or
  status composition (see stakeholder-update).
---

# Meeting Notes & Actions

## Overview

Notes are a filter, not a recording: three payloads — decisions with their whys, actions with the owner-verb-date triple, open questions — and everything else released. The readback is the two-minute quality gate; the next meeting's opening three minutes on last time's actions is the enforcement that converts capture into completion.

## When to Use

- Any meeting that produces decisions or commitments (producing neither questions the meeting).
- A team's recurring pattern: the same discussion re-had monthly because nothing was captured.

**When NOT to use:**
- Personal task systems — `weekly-review` absorbs the outputs.
- Status updates — `stakeholder-update`.

## Prerequisites

- Note-taking responsibility assigned BEFORE the meeting (rotating is fine; ambient "someone will" produces nobody), and a searchable home for the notes.

## The Workflow

1. **Capture three things live; let everything else go:** decisions (what, by whom, including options explicitly rejected — the rejection record prevents the re-litigation), actions (step 2's format), open questions (unresolved, needing owners). The transcript instinct buries the payloads in prose nobody re-reads — and AI transcription changes nothing: the machine transcript still needs THIS distillation, and auto-extracted "action items" still need step 2's negotiation.

2. **Actions get the non-negotiable triple: owner + verb-object + date.** "Sam: send the revised pricing doc to legal — by Thu." One named owner (a shared action is an orphaned action); a concrete deliverable; a date extracted in the room (ten seconds of awkwardness now vs a week of ambiguity later). The negotiation matters: an owner ACCEPTING aloud is the commitment mechanism — items assigned to absentees or slipped in un-negotiated complete at near-zero rates.

3. **Record decisions with their one-line why:** "Decided: launch slips to March 3 — compliance review can't complete before Feb 20 (rejected: launching with the EU flag off — support burden too high)." The why is what makes the record useful when circumstances shift: a decision with its reasoning can be re-evaluated when the reason dies; a bare decision ossifies or gets silently re-decided. Big decisions graduate to `architecture-decision-record`, linked.

4. **Close the meeting with the 2-minute readback:** actions and decisions read aloud in the final minutes — the cheapest quality gate in the genre, routinely catching "that's not what I agreed to," missing owners, and the decision half the room heard differently. Objections now cost two minutes; objections next week cost the meeting again.

5. **Distribute within hours, in scannable form:** decisions and actions AT THE TOP, posted to the agreed home and pushed where participants live — same day, while memory can still correct the record. Notes three days later are archaeology; notes never distributed are theater. Owners tagged.

6. **Run the follow-through loop — the step that makes any of this matter:** open actions in one visible place per team, and the NEXT meeting of the series opens with 3 minutes on the last one's actions (done / on-track / renegotiated). An action missed twice gets renegotiated honestly (real deadline, real owner, or deliberate drop) — zombie actions rolling forward train everyone that commitments here are decorative.

7. **Feed the downstream consumers:** owners' items land in personal systems (`weekly-review`'s sweep collects them); decisions affecting outsiders get carried to them deliberately (the decision nobody told the affected team is a classic incident seed); and the searchable home pays for the whole discipline the first time "didn't we decide this in Q1?" is answered with a link instead of an hour.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "I capture everything — full context preserved" | Four faithful paragraphs with the decision implicit in paragraph three is unfindable and unactionable. The filter IS the value; transcripts are what recordings are for. |
| "'We should update the runbook' — noted as an action" | Ownerless actions belong to nobody and complete at nobody's rate. The triple isn't formatting; the in-room acceptance is the commitment mechanism itself. |
| "Asking 'by when?' in the meeting feels pushy" | Ten seconds of mild awkwardness now, or a week of 'when I get to it' later. Dateless actions read as optional because, functionally, they are. |
| "The readback wastes the last two minutes" | It catches 'that's not what I agreed to' while correction costs nothing. Skipped, two attendees leave with different decisions — discovered at the next meeting, at full price. |
| "AI transcription handles the notes now" | The transcript is raw material; the distillation (three payloads), the owner negotiation, and the readback are exactly the parts the machine doesn't do. Automation moved the work; it didn't do it. |
| "We captured the actions — job done" | Captured-and-never-revisited is elaborate stenography. The next-meeting 3-minute review is where capture converts to completion; without it, zombie actions teach everyone the ritual is hollow. |

## Red Flags

- The same topic re-decided across multiple meetings, no record cited.
- Action lists without owners or dates.
- Notes distributed days later, or to a wiki nobody opens.
- No readback; post-meeting disagreement about what was decided.
- Actions rolling forward meeting after meeting, unrenegotiated.
- Decisions affecting other teams that those teams learn about by collision.

## Verification

- [ ] Note-taker named in the invite; template with the three payloads at top.
- [ ] Every action carries the triple; owners accepted in-room — visible in the notes.
- [ ] Decisions carry whys and rejected alternatives — spot-check.
- [ ] Readback done — noted; corrections incorporated.
- [ ] Distribution same-day to the agreed home + participants' channel, owners tagged — link.
- [ ] The series' next meeting opened with the action review — recurring agenda item shown; zombie count zero (twice-missed items explicitly renegotiated or dropped).

## Example

Weekly product-eng sync, 8 people, textbook symptoms: the API-versioning discussion re-had three times in six weeks (each "decided," nothing recorded), and a launch-blocking legal review "someone was handling" for a month. Practice installed: rotating note-taker in the invite, template with Decisions/Actions/Open-questions at top. First meeting under the regime: the versioning decision captured WITH its rejected alternative and why; four actions with the triple, one of which surfaced the legal-review orphan — the readback caught that the presumed owner thought legal owned it (the exact mechanism of its month adrift; owner negotiated in-room, dated Friday). Distribution same afternoon. Next sync's opening 3 minutes: 3 of 4 done, the fourth renegotiated aloud. Six weeks in: the versioning question resurfaced from a new joiner — answered with a link in 30 seconds; zombie actions: zero, because the twice-missed rule dropped one item explicitly ("deliberately dropped: the migration guide — superseded by the docs rewrite"). Meeting length: unchanged. What changed was that its outputs started existing.

## Related skills

- `weekly-review` — where owners' actions get absorbed and chased.
- `architecture-decision-record` — the graduation path for big decisions.
- `stakeholder-update` — fed by the decision/action stream.
- `kpi-reporting-pack` — the same opening-with-last-actions enforcement device.
