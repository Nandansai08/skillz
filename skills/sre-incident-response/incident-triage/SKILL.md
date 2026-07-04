---
name: incident-triage
description: >
  Use in the first minutes of a suspected production incident — assessing
  severity, declaring, assigning roles, stabilizing before diagnosing.
  Triggers: "production is down", "we have an incident", "users are
  reporting errors", "should we declare an incident", "sev1/sev2", "site
  is slow". NOT for the deep diagnosis itself (see production-debugging)
  or the writeup after (see blameless-postmortem).
---

# Incident Triage

## Overview

The first fifteen minutes decide most of an incident's cost: impact assessed in user terms, severity declared without ego, roles split so someone coordinates while others type, and mitigation chosen over diagnosis. Structure is what replaces adrenaline.

## When to Use

- An alert fired, users are complaining, or a deploy went sideways — and the next 15 minutes need structure.
- Deciding whether something *is* an incident.

**When NOT to use:**
- Root-cause analysis during or after — `production-debugging`.
- The post-resolution writeup — `blameless-postmortem`.

## Prerequisites

- Access to dashboards, deploy history, and the incident channel/tooling. (If these don't exist, the first postmortem action item writes itself.)

## The Workflow

1. **Assess impact in user terms, fast and rough.** Who can't do what, since when, how many? "Checkout failing for ~all EU users since 14:02" — three data points, two minutes, decides everything downstream. Don't debug yet.

2. **Declare using the severity ladder — when unsure, declare high and downgrade.**
   - **SEV1:** core user journey down or data being lost/corrupted, no workaround. Page everyone needed, exec visibility.
   - **SEV2:** major degradation or a core journey down for a subset; workaround exists.
   - **SEV3:** minor feature broken, perf degraded but usable — business hours response.
   Declaring is cheap; a late declaration costs the timeline, the help, and the trust. Nobody gets criticized for a downgrade.

3. **Assign the two roles immediately.**
   - **Incident Commander (IC):** coordinates, decides, communicates. Explicitly *not* the person typing into terminals — the moment the IC starts debugging, coordination stops.
   - **Ops lead(s):** hands on keyboards.
   Small team? IC + one responder is still two hats on two heads. Add a comms role at SEV1.

4. **Stabilize before diagnosing — mitigation first.** The question is "what stops the bleeding," not "what's the root cause." Check the levers in order of speed:
   - Recent deploy in the window? **Roll back now** (see `deployment-rollback-plan`) — don't wait to confirm it's the cause; rollback is cheap, downtime isn't.
   - Feature flag covering the broken path? Kill it.
   - One bad node/AZ/dependency? Drain, failover, or degrade gracefully.
   One change at a time — simultaneous changes mean recovery can't be attributed and secondary breakage can't be isolated.

5. **Start the timeline log from minute one.** Timestamped notes in the incident channel: what's observed, what's tried, what changed. Two purposes: joiners onboard by reading instead of interrupting, and the postmortem writes itself. "14:09 rolled back api to v512 — error rate unchanged" is the format.

6. **Communicate on a clock, not on progress.** SEV1: update every 15–30 min *even if the update is "still investigating."* Silence reads as abandonment and generates the exec-pinging-the-IC loop. Status page for users when external impact exceeds a few minutes.

7. **Close deliberately:** confirm recovery in the *user-facing* metric (not just the server metric), watch one full recovery period for relapse, post the all-clear, downgrade or resolve — and schedule the postmortem before everyone scatters (within 48h while memory is fresh).

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "Let's find the root cause first, then fix it properly" | Forty minutes of elegant diagnosis while a rollback would have restored users in minute three. Mitigation first is the whole doctrine; understanding waits happily. |
| "It might not be the deploy — rolling back without proof is unscientific" | Rollback costs minutes and is reversible; downtime is neither. Roll back on the correlation; confirm causation from a working system. |
| "I'll IC and debug at the same time — hands are short" | The moment the IC opens a terminal, updates stop, joiners flounder, and two engineers duplicate changes. Two hats, two heads, even on a team of two. |
| "Declaring SEV1 for this will look like panic" | Under-declaring costs the timeline, the pre-warmed help, and everyone's trust when it escalates late. The ladder explicitly prices downgrades at zero. |
| "The fix is one line — faster to fix forward than roll back" | The one-line fix written at 2am without review is a well-documented second incident. Roll back first; fix forward calmly tomorrow. |
| "No update yet — nothing new to report" | Silence reads as abandonment and spawns the exec-ping loop that eats the IC alive. 'Still investigating, next update 14:45' IS the update. |

## Red Flags

- Five engineers debugging in five directions, nobody posting to the channel.
- Multiple mitigation changes applied within the same minutes.
- First status update more than 30 minutes after detection.
- Root-cause discussion dominating the channel while users are still down.
- Incident "resolved" on a server metric while the user-facing metric hasn't recovered.
- No postmortem scheduled a week after a SEV1.

## Verification

- [ ] Impact statement (who/what/since when/how many) posted within minutes — timestamped in channel.
- [ ] Severity declared and roles named (IC + ops) in the channel — visible.
- [ ] Mitigation attempted before deep diagnosis — timeline shows the order.
- [ ] One-change-at-a-time held — timeline shows attribution per change.
- [ ] Recovery confirmed in the user-facing metric, relapse window observed — metric link in the all-clear.
- [ ] Postmortem scheduled within 48h — calendar/ticket link.

## Example

14:02 alert: checkout error rate 40%. IC declared SEV1 by 14:06 (core journey, all users). Roles: Dana IC, Marco ops. 14:08 deploy history shows payments-service shipped 13:55 — rollback initiated *before* any diagnosis, done 14:12. Error rate normal 14:13; user-facing conversion metric recovered by 14:20; watched until 14:45, all-clear posted. Timeline log had 9 entries; postmortem scheduled for next day found the actual bug (connection-pool config) in 30 calm minutes. Total user impact: 11 minutes — of which zero were spent root-causing during the incident.

## Related skills

- `production-debugging` — the diagnosis discipline, during (if mitigation fails) or after.
- `deployment-rollback-plan` — why step 4's rollback was ready to execute.
- `blameless-postmortem` — the mandatory follow-through.
- `runbook-authoring` — pre-writing step 4's levers per service.
