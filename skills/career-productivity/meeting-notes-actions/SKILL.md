---
name: meeting-notes-actions
description: >
  Use when capturing meetings so decisions and actions actually survive —
  owner-assigned action items, decision records, distribution and
  follow-through mechanics. Triggers: "take notes for this meeting",
  "turn these notes into action items", "nobody remembers what we decided",
  "meeting minutes", "actions from the sync".
---

# Meeting Notes & Actions

## When to use this skill
- Any meeting that produces decisions or commitments (if it produces neither, question the meeting).
- A team's recurring pattern: same discussion re-had monthly because nothing was captured.
- NOT for personal task management (`weekly-review` consumes this skill's output) or status-update composition (`stakeholder-update`).

## Prerequisites
- Note-taking responsibility assigned BEFORE the meeting starts (rotating is fine; ambient "someone will" produces nobody), and a consistent home for the notes (channel, wiki page series — searchable, not private files).

## Workflow

1. **Capture three things live; let everything else go:** decisions (what was decided, by whom, including options explicitly rejected — the rejection record is what prevents the re-litigation), actions (see step 2's format), and open questions (raised, unresolved, needing an owner). The transcript instinct — capturing discussion verbatim — buries the three payloads in prose nobody re-reads; notes are a filter, not a recording (AI transcription changes nothing here: the machine transcript still needs THIS distillation pass, and auto-extracted "action items" still need step 2's negotiation).

2. **Actions get the non-negotiable triple: owner + verb-object + date.** "Sam: send the revised pricing doc to legal — by Thu." One named owner (a shared action is an orphaned action — split it or name a lead); a concrete deliverable (not "look into pricing"); a date extracted in the room (the 10 seconds of "by when?" awkwardness now versus the week of ambiguity later). The negotiation matters: an owner ACCEPTING the action aloud is the commitment mechanism — items assigned to absent people or slipped into notes un-negotiated have near-zero completion rates.

3. **Record decisions with their one-line why:** "Decided: launch slips to March 3 — the compliance review can't complete before Feb 20 (rejected: launching with the EU flag off — support burden judged too high)." The why-line is what makes the record useful in six weeks when circumstances shift: a decision with its reasoning can be re-evaluated when the reason dies; a bare decision either ossifies or gets silently re-decided (`architecture-decision-record` at meeting scale — big decisions graduate to real ADRs, and the note says so with a link).

4. **Close the meeting with the 2-minute readback:** actions and decisions read aloud in the final minutes — the cheapest quality gate in the genre, routinely catching "that's not what I agreed to," missing owners, and the decision half the room heard differently. Silence after readback = the record stands; objections now cost two minutes, objections next week cost the meeting again.

5. **Distribute within hours, in scannable form:** decisions and actions AT THE TOP (context below for the curious), posted to the agreed home and pushed to the channel where participants live — same day, while memory can still correct the record. Notes distributed three days later are archaeology; notes never distributed are theater. Tag the owners on their items.

6. **Run the follow-through loop — the step that makes any of this matter:** open actions tracked in one visible place per team (the notes' actions flow into it — a tracker, a pinned doc, whatever's actually looked at), and the NEXT meeting of the same series opens with 3 minutes on the last one's actions (done / on-track / renegotiated — the recurring-agenda slot is the enforcement mechanism; `kpi-reporting-pack` step 5 uses the identical device). An action missed twice gets renegotiated honestly (real deadline, real owner, or deliberate drop) — zombie actions rolling forward meeting after meeting train everyone that commitments here are decorative.

7. **Feed the downstream consumers:** owners' items land in their personal systems (`weekly-review` step 2's sweep collects them); decisions that affect outsiders get carried to them deliberately (the decision nobody told the affected team about is a classic incident seed); and the notes' searchable home earns its keep the first time "wait, didn't we decide this in Q1?" is answered with a link instead of an hour of relitigating (the payoff that funds the whole discipline).

## Common pitfalls
- Transcript-notes: four paragraphs of faithful discussion, the decision implicit somewhere in paragraph three — unfindable, unactionable, unread (step 1's filter).
- Ownerless actions: "we should update the runbook" — recorded, distributed, and belonging to nobody. The owner-negotiation (step 2) is the entire difference between a note and a commitment.
- Dateless actions: owned but unbounded — "when I get to it" is the honest reading, and it's usually accurate.
- The skipped readback: two attendees leave with different decisions, discovered at the next meeting — two minutes of prevention declined (step 4).
- Notes into the void: captured beautifully, distributed nowhere, or to a wiki nobody opens — distribution to where people LIVE (step 5), or the capture was ceremony.
- No follow-through loop: actions captured perfectly and never revisited — the tracker and the opening-3-minutes (step 6) are where capture converts to completion; without them the whole practice is elaborate stenography.

## Example
Weekly product-eng sync, 8 people, textbook symptoms: the API-versioning discussion re-had three times in six weeks (each time ~20 minutes, each time "decided," nothing recorded), and a launch-blocking legal review that "someone was handling" for a month. Practice installed: rotating note-taker named in the invite, template with Decisions/Actions/Open-questions at top. First meeting under the regime: the versioning decision captured WITH its rejected alternative and why ("URL versioning; rejected header-based — client-cache complications"); four actions with the triple, one of which surfaced the legal-review orphan — readback caught that the presumed owner thought legal owned it (the exact mechanism of its month adrift; owner negotiated in-room, dated Friday). Distribution same afternoon, owners tagged. The next sync's opening-3-minutes: 3 of 4 done, the fourth renegotiated aloud. Six weeks in: the versioning question resurfaced from a new joiner — answered with a link in 30 seconds (the re-litigation that didn't happen); zombie actions: zero, because the twice-missed rule dropped one item explicitly ("deliberately dropped: the migration guide — superseded by the docs rewrite") instead of rolling it. Meeting length: unchanged. What changed was that its outputs started existing.

## Related skills
- `weekly-review` — where owners' actions get absorbed and chased.
- `architecture-decision-record` — the graduation path for big decisions.
- `stakeholder-update` — fed by the decision/action stream.
- `kpi-reporting-pack` — the same opening-with-last-actions enforcement device.
