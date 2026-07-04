---
name: on-call-handoff
description: >
  Use when transferring on-call duty between people — structured handoff
  of open incidents, ongoing risks, and context so nothing drops between
  shifts. Triggers: "on-call handoff", "end of my rotation", "taking over
  on-call", "shift change", "what do I need to know for on-call this
  week". NOT for mid-incident IC transfer — that's a live role transfer
  inside incident-triage (announce, brief on the timeline, confirm
  acceptance).
---

# On-Call Handoff

## Overview

Dropped balls between shifts live in the gaps: the untracked waiting-for, the silence with no expiry, the access that rotted since last rotation. A fixed-template note plus five synchronous minutes converts shift change from a vibe into a transfer of state.

## When to Use

- Ending or starting an on-call rotation (weekly, daily, or follow-the-sun).
- Designing a team's handoff ritual from scratch.

**When NOT to use:**
- Mid-incident IC transfer — a live handoff inside `incident-triage`: announce, brief on the timeline log, confirm acceptance. One paragraph, not a document.

## Prerequisites

- A persistent, searchable location for handoff notes (channel thread, wiki page series) — verbal-only handoffs are goldfish memory.
- Five minutes of synchronous overlap when anything nontrivial is open.

## The Workflow

1. **Write the handoff note against a fixed template — same sections every week:**
   - **Open incidents:** state, severity, next expected action, links to timeline.
   - **Smoldering risks:** things not-yet-incidents — "disk on db-3 at 78% and climbing ~2%/day", "vendor X flaky since Tuesday, retries absorbing it", "deploy of Y went out Friday, first weekend under load."
   - **Recent changes:** deploys/migrations/flag flips whose risk hasn't fully soaked.
   - **Alert weather:** which alerts have been noisy, which are newly added, any silences currently active *with expiry* — an inherited silence with no expiry is a detection hole.
   - **Follow-ups owed:** tickets promised, postmortems scheduled, customers awaiting response.
   Fixed structure matters: the reader learns where to look, and an empty section is information ("no open incidents") instead of ambiguity.

2. **Rank items by "probability it pages you × how lost you'd be."** The note leads with the thing most likely to fire, not chronology. One line each; links carry the depth.

3. **Hand off state, not just facts.** For each open item: what's been tried, what's been ruled out, what the current hypothesis is. The incoming person repeating three hours of eliminated hypotheses is the classic handoff failure.

4. **Do the synchronous pass as a walkthrough, incoming person driving:** they read the note, ask questions, and *confirm access* — pager routed correctly (send a test page on rotation change), VPN/kubeconfig/dashboard access alive, escalation contacts current. Access rot is a handoff-day discovery, never an incident-day one.

5. **Explicitly transfer or park each open item.** Every open incident/follow-up gets one of: transferred (incoming owns it), retained (outgoing keeps this one item — with the incoming person's knowledge), or parked (ticketed, nobody's holding it, deliberately). Items in none of these states are the ones that vanish.

6. **Incoming person's first hour:** skim the last two handoff notes (not just one — trends live across weeks: "disk at 74%" last week + "78%" this week is a trajectory), check the current dashboards to baseline "normal for now", and note the current top risk in your own words back to the outgoing person — the echo test catches misunderstanding immediately.

7. **Feed the rotation's exhaust into the system:** recurring smoldering-risk entries (three consecutive weeks of "vendor X flaky") escalate to real tickets; noisy-alert notes feed the monthly alert review (`alerting-design` step 7); handoff friction ("I couldn't find the runbook for Z") becomes runbook backlog. The handoff note is also the team's best sensor for slow-burn problems.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "Quiet week — nothing to hand off" | With a silence active, a migration half-soaked, and disk climbing 2%/day. The template's sections force the memory; 'quiet' is what unexamined weeks feel like. |
| "I'll stay reachable — no need for formal transfer of my incident" | Two people each assuming the other is watching is zero people watching. Retained-with-knowledge is fine; informal shadow-ownership is how it resurfaces as a surprise. |
| "The next person can read the incident channel if they're curious" | Three hundred messages don't transfer a hypothesis state. The note's 'tried/ruled out/current theory' line saves the incoming person from re-running your eliminations at 3am. |
| "Access check is paranoia — it worked last rotation" | Tokens expire, SSO lapses, escalation contacts change teams. The test page costs 30 seconds at handoff; discovering the broken route costs 30 minutes into an outage. |
| "A verbal walkthrough is warmer than a document" | And unsearchable on Wednesday when the pager fires. The doc is the artifact; the sync pass is the warmth — both, not either. |
| "Mentioning the silence I set feels like admitting a mess" | The inherited silence with no expiry is a detection hole the incoming person operates inside blind. Alert-weather honesty is the section that prevents invisible gaps. |

## Red Flags

- Handoffs happening only verbally, or not at all on "quiet" weeks.
- Active silences/downtimes not mentioned in any note.
- The incoming person's first kubectl of the week failing on auth — during an incident.
- The same smoldering risk appearing three weeks running with no ticket.
- Open items with ambiguous ownership after the handoff.
- Notes written but the sync walkthrough skipped whenever things are busy (i.e., exactly when it matters).

## Verification

- [ ] Note posted in the persistent home, all five sections present (empty ones marked "none") — link.
- [ ] Each open item explicitly transferred/retained/parked — states visible in the note.
- [ ] Test page sent and received on rotation change — confirmed in the note.
- [ ] Access checklist passed (pager, VPN, kubeconfig, dashboards, escalation contacts) — checked off.
- [ ] Incoming person's echo test done (top risk restated) — one line in the thread.
- [ ] Recurring risks (3+ weeks) escalated to tickets — links.

## Example

Rotation change, Friday. Note led with: (1) smoldering — payment retries absorbing vendor flakiness since Wed, "if retry queue > 5k, vendor is fully down, runbook link, account manager contact"; (2) silence on the noisy cert alert, expires Monday; (3) Friday's checkout deploy, first weekend under load, rollback command pre-staged; (4) parked: postmortem doc due Tue. Sync walkthrough: test page delivered, incoming discovered their Grafana SSO had lapsed (fixed in 5 min, not at 3am). Saturday the vendor did fully fail — incoming person recognized it from the note in one minute, executed the linked runbook, called the pre-listed contact. Monday's review escalated the three-week vendor pattern into a ticketed migration.

## Related skills

- `incident-triage` — mid-incident IC transfer (the live variant).
- `alerting-design` — where the alert-weather observations flow.
- `meeting-notes-actions` — the same owner-or-it-vanishes discipline, general form.
