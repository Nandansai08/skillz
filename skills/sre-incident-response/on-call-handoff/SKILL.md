---
name: on-call-handoff
description: >
  Use when transferring on-call duty between people — structured handoff of
  open incidents, ongoing risks, and context so nothing drops between shifts.
  Triggers: "on-call handoff", "end of my rotation", "taking over on-call",
  "shift change", "what do I need to know for on-call this week".
---

# On-Call Handoff

## When to use this skill
- Ending or starting an on-call rotation (weekly, daily, or follow-the-sun).
- Designing a team's handoff ritual from scratch.
- NOT for mid-incident IC transfer — that's a live role transfer inside `incident-triage` (announce, brief on the timeline log, confirm acceptance — one paragraph, not a document).

## Prerequisites
- A persistent, searchable location for handoff notes (channel thread, wiki page series) — verbal-only handoffs are goldfish memory.
- Five minutes of synchronous overlap when anything nontrivial is open (async-only works only for quiet weeks).

## Workflow

1. **Write the handoff note against a fixed template — same sections every week:**
   - **Open incidents:** state, severity, next expected action, links to timeline.
   - **Smoldering risks:** things not-yet-incidents — "disk on db-3 at 78% and climbing ~2%/day", "vendor X flaky since Tuesday, retries absorbing it", "deploy of Y went out Friday, first weekend under load."
   - **Recent changes:** deploys/migrations/flag flips in the window whose risk hasn't fully soaked.
   - **Alert weather:** which alerts have been noisy, which are newly added, any silences currently active *with expiry* — an inherited silence with no expiry is a detection hole.
   - **Follow-ups owed:** tickets promised, postmortems scheduled, customers awaiting response.
   Fixed structure matters: the reader learns where to look, and an empty section is information ("no open incidents") instead of ambiguity.

2. **Rank items by "probability it pages you × how lost you'd be."** The note leads with the thing most likely to fire, not chronology. One line each; links carry the depth.

3. **Hand off state, not just facts.** For each open item: what's been tried, what's been ruled out, what the current hypothesis is. The incoming person repeating three hours of eliminated hypotheses is the classic handoff failure (`production-debugging` step 5's discipline, transferred).

4. **Do the synchronous pass as a walkthrough, incoming person driving:** they read the note, ask questions, and *confirm access* — pager routed correctly (send a test page on rotation change; wrong-routing is discovered otherwise at 3am), VPN/kubeconfig/dashboard access alive, escalation contacts current. Access rot is a handoff-day discovery, never an incident-day one.

5. **Explicitly transfer or park each open item.** Every open incident/follow-up gets one of: transferred (incoming owns it), retained (outgoing keeps this one item — with the incoming person's knowledge), or parked (ticketed, nobody's holding it, deliberately). Items in none of these states are the ones that vanish.

6. **Incoming person's first hour:** skim the last two handoff notes (not just one — trends live across weeks: "disk at 74%" last week + "78%" this week is a trajectory), check the current dashboards to baseline "normal for now", and note the current top risk in your own words back to the outgoing person — the echo test catches misunderstanding immediately.

7. **Feed the rotation's exhaust into the system:** recurring smoldering-risk entries (three consecutive weeks of "vendor X flaky") escalate to real tickets; noisy-alert notes feed the monthly alert review (`alerting-design` step 7); handoff friction ("I couldn't find the runbook for Z") becomes runbook backlog. The handoff note is also the team's best sensor for slow-burn problems.

## Common pitfalls
- "Quiet week, nothing to hand off" — with a silence active, a migration half-done, and disk climbing. Quiet weeks still fill the template; the sections force the memory.
- Handoff as unstructured war story: twenty minutes of narrative, no artifact, nothing searchable when the pager fires Wednesday.
- Silences and downtimed alerts not mentioned. The incoming person operates blind exactly where the outgoing person knew to worry.
- Outgoing person keeps informal ownership of "their" incident without saying so — two people each assuming the other is watching. Step 5's explicit states exist for this.
- Access checked never: the new on-call's first kubectl command of the week is during an outage, and their token expired two rotations ago.
- Handoff notes written but never read — pair the artifact with the 5-minute sync; the walkthrough is what forces the read.

## Example
Rotation change, Friday. Note led with: (1) smoldering — payment retries absorbing vendor flakiness since Wed, "if retry queue > 5k, vendor is fully down, runbook link, account manager contact"; (2) silence on the noisy cert alert, expires Monday; (3) Friday's checkout deploy, first weekend under load, rollback command pre-staged; (4) parked: postmortem doc due Tue. Sync walkthrough: test page delivered, incoming discovered their Grafana SSO had lapsed (fixed in 5 min, not at 3am). Saturday the vendor did fully fail — incoming person recognized it from the note in one minute, executed the linked runbook, called the pre-listed contact. Monday's review escalated the three-week vendor pattern into a ticketed migration.

## Related skills
- `incident-triage` — mid-incident IC transfer (the live variant).
- `alerting-design` — where the alert-weather observations flow.
- `meeting-notes-actions` — the same owner-or-it-vanishes discipline, general form.
