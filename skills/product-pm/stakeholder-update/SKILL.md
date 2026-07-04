---
name: stakeholder-update
description: >
  Use when writing recurring status updates for stakeholders — progress,
  risks, and asks on one screen, readable in 90 seconds, honest about
  trouble early. Triggers: "status update for", "weekly update", "exec
  update", "project status report", "keep leadership informed". NOT for
  incident communications (see incident-triage's clock-driven cadence) or
  roadmap presentations (see roadmap-communication).
---

# Stakeholder Update

## Overview

Updates exist to make surprises impossible: status-against-goal in the first line, bad news the week it becomes *likely*, and asks that are named, dated, and aimed at someone who can act. The metronome cadence is itself the trust signal — skipped updates read as hiding exactly when things are hard.

## When to Use

- A project needs recurring status communication to people not in its daily flow.
- Updates exist but nobody reads them, or leadership keeps getting surprised anyway.

**When NOT to use:**
- Incident comms — `incident-triage` step 6.
- Roadmap presentations — `roadmap-communication`.

## Prerequisites

- Clarity on who reads this and what decisions they can make — the asks section aims at THEIR powers.

## The Workflow

1. **Lead with status-against-goal in one line, color-coded honestly:** "🟡 Checkout revamp — trending 1 week late on the June 30 target; mitigation in flight (below)." The reader's first question is "should I worry?"; answer it in ten words. Color semantics published once: green = on track; yellow = at risk, mitigation exists; red = will miss without intervention — *red means "I need something", not "I failed"*, and the team's culture around that sentence determines whether you ever get true yellows.

2. **Structure the body as Progress / Risks / Asks — one screen, always the same shape:**
   - **Progress:** 3–5 outcome bullets tied to the goal ("payment-provider integration passing E2E" — not "had productive syncs"). A number where one exists.
   - **Risks:** each with trajectory and owner ("vendor API latency 2× spec — testing workaround, decision by Fri — @sam"). A risk repeated three updates unchanged is stalled: escalate or close it.
   - **Asks:** specific, addressed, dated ("need staffing decision on the second backend eng by Wed — blocks the July milestone"). No asks for three straight updates usually means unasked asks.

3. **Surface bad news early, framed as trajectory + plan:** the update where slippage first *appears* should be the one where it first *became likely* — not after it's certain. "If this week's fix fails, June 30 slips ~1 week; decision point Friday" costs a moment of discomfort and buys the thing updates exist for. Bad news aged two weeks converts from information into betrayal.

4. **Write for the skimming reader:** bold the load-bearing phrases, one screen hard limit, links for depth, zero project-internal jargon without gloss — the exec forwarding your update to THEIR boss is the real audience.

5. **Keep the goal visible and version its changes:** restate goal + date in the header every time; if the goal itself changed, that's the headline with the reason — a goal quietly restated to match reality is the most corrosive update pattern there is.

6. **Make the cadence self-enforcing:** same day, same channel, every cycle — a skipped update reads as "hiding something" precisely when things are hard; a running-notes file (`weekly-review`, `meeting-notes-actions` feed it) makes the update 15 minutes of curation, not an hour of archaeology.

7. **Audit quarterly with two questions:** did any reader get surprised by something this stream should have carried? (each surprise = a step-3 failure to fix); are the asks getting answered? (unanswered = wrong audience, wrong specificity, or unread — diagnose which; an unread update is a writing problem before it's a reader problem).

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "We did a lot this week — list the activities" | 'Met with design, reviewed PRs' is motion without position. Readers need where-we-are-versus-goal; the activity log answers a question nobody asked. |
| "It might resolve itself — report it next week if not" | The watermelon pattern: green outside, red inside, detonating a week before the deadline. One honest yellow now costs discomfort; the aged surprise costs the project's credibility and yours. |
| "I mentioned the blocker in paragraph four" | Buried asks don't get answered, and then 'leadership never unblocks us' becomes the retro theme. Asks are the payload: own section, named person, date. |
| "Quiet week — skip the update" | Skips train readers that silence is ambiguous, and the skip always lands the week things got hard. 'Steady, no change' takes one line and keeps the metronome's trust. |
| "One update serves execs and the working team" | One drowns, the other starves. Same source notes, two texts — the compression is the work, and it's 10 minutes with the running file. |
| "The goal moved — I'll just update the header quietly" | The silently-moving goalpost is the most corrosive pattern in the genre. Goal changes are headlines with reasons, or every future green is suspect. |

## Red Flags

- Status colors that have never shown yellow on a project that struggled.
- Activity lists where outcomes should be.
- The same risk verbatim across three updates.
- Asks without names or dates; asks buried in prose.
- Updates arriving on random days, or not at all in hard weeks.
- A reader learning about a slip in a meeting instead of the update.

## Verification

- [ ] First line carries status + goal + trajectory — check the last three updates.
- [ ] Fixed Progress/Risks/Asks shape, one screen — verified.
- [ ] Every ask has addressee + date; answer-rate tracked — last quarter's asks reviewed.
- [ ] Slippage surfaced at likely, not certain — timeline of the last yellow examined.
- [ ] Cadence unbroken (or degraded one-liners on hard weeks) — history shown.
- [ ] Quarterly surprise-audit done — zero reader surprises, or fixes named.

## Example

Platform-migration project, biweekly exec update, previously a 3-page prose email nobody confirmed reading. Rebuilt on the template: header restated goal+date; status 🟡 in week 4 the moment the data-backfill rate first trended 40% under plan (flagged at *likely*, not certain), with the Friday decision point named. The ask ("need the DBA on loan for two weeks — blocks cutover rehearsal") addressed to the one exec who could approve it, dated — approved in a day, versus the prior project where the same ask lived unnoticed in paragraph six for a month. Week 6: fix landed, back to 🟢, one line. Post-project audit: zero exec surprises across 14 updates (the prior migration had produced three escalations from surprise alone); the running-notes file doubled as the retro's timeline. Writing cost per update after the template settled: ~20 minutes.

## Related skills

- `roadmap-communication` — the strategic-horizon sibling.
- `meeting-notes-actions` — the raw material feed.
- `incident-triage` — the crisis-mode communication variant.
- `weekly-review` — the personal practice that makes updates cheap.
