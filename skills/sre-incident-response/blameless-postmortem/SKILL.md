---
name: blameless-postmortem
description: >
  Use after an incident is resolved to write the postmortem — timeline,
  contributing factors, action items that actually get done, without
  blame. Triggers: "write the postmortem", "incident review", "RCA
  document", "post-incident review", "lessons learned from the outage".
  NOT during the incident (see incident-triage) — this starts once users
  are restored.
---

# Blameless Postmortem

## Overview

Incidents are tuition; the postmortem is whether you keep the learning. Blamelessness isn't kindness policy — it's data quality: the moment postmortems punish, people edit timelines and stop reporting near-misses, and the sensor network goes dark.

## When to Use

- After any SEV1/SEV2, near-misses included (near-misses are free postmortems).
- Reviewing someone else's postmortem draft.

**When NOT to use:**
- During the incident — `incident-triage`; this starts once users are restored.

## Prerequisites

- The incident timeline log (channel history, deploy records, alert timestamps).
- Scheduled within 48h of resolution — memories of "why did we think that?" decay fast.
- The template: [templates/postmortem.md](templates/postmortem.md).

## The Workflow

1. **Reconstruct the timeline from artifacts, not memory.** Timestamps from: alerts, deploys, chat messages, dashboard annotations, commands run. Include the *detection gap* (impact start → first human awareness) and *decision points* ("14:20: considered rollback, chose fix-forward because X") — the decisions are where the learning lives. Start the timeline BEFORE detection: the change that armed the trap days earlier belongs in it.

2. **Write impact in numbers:** duration, users/requests affected, revenue/SLA/error-budget cost, support tickets. "Bad outage" is not analyzable; "31 min, 12% of checkouts failed, ~$40k GMV, 60% of monthly error budget" is.

3. **Hunt contributing factors, not THE root cause.** Real incidents are 3–6 factors aligning. Use "why did the system allow this?" not "who did this?" — every time the answer is a person's action, ask what made that action reasonable and undetectable:
   - ~~"Dev pushed a bad config"~~ → config had no validation, no canary, and review couldn't see the generated output.
   Categories to probe: the trigger, why defenses missed it (tests, review, canary), why detection was slow, why mitigation was slow, what amplified it.

4. **Enforce blamelessness structurally.** Names replaced by roles in the doc. The test: could every participant hand this doc to their manager without flinching? Watch for the hindsight contaminant: every "we should have known/checked" gets reframed as "what would have made this visible *at the time*?"

5. **Write action items that survive contact with the backlog.** Each one: specific, owned by a name, ticketed, deadlined, and sized honestly. Split into:
   - **Prevent recurrence** (fix the class, not the instance — validation, guardrail, removal of the trap);
   - **Detect faster** (the alert that was missing — see `alerting-design`);
   - **Mitigate faster** (runbook, kill switch, drill).
   Three excellent items beat twelve aspirational ones; "improve testing culture" is not an action item, and neither is anything assigned to "the team."

6. **Extract the transferable lesson.** Ask: where *else* does this pattern exist? (Same config system, same missing canary, sibling services.) The instance is fixed; the class is the payoff — one grep or audit action item usually covers it.

7. **Review the follow-through, not just the doc.** 30 days later: are the action items done? Postmortem programs die not from bad docs but from action items rotting — track completion rate as a team metric, and re-raise stale items in the next review.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "Root cause: human error — case closed" | Humans err at a constant rate; systems decide whether errors become outages. 'Human error' means the analysis stopped one 'why' too early, guaranteed. |
| "We all know what happened, the doc is a formality" | What everyone 'knows' diverges within a week, and the action items nobody wrote down complete at the rate of zero. The doc is the mechanism, not the memento. |
| "It was a near-miss, nothing actually broke — skip it" | A near-miss is the identical lesson at zero user cost — the cheapest postmortem you'll ever run. Skipping them means learning only from the expensive ones. |
| "Obviously we should have tested that — write that down" | Hindsight counterfactuals contaminate the analysis: the question is why the knowledge wasn't available AT THE TIME. Otherwise every postmortem concludes 'be smarter,' which changes nothing. |
| "Twelve action items shows we're taking it seriously" | Twelve aspirational items complete at ~zero; three owned, dated, sized ones complete. Volume is how seriousness gets performed instead of practiced. |
| "The postmortem is done — the doc looks great" | Postmortem theater: beautiful doc, rotting action items, same incident in Q3. The 30-day completion check is the difference between ritual and practice. |

## Red Flags

- The phrase "human error" anywhere in the factors section.
- A timeline that starts at detection (the armed trap invisible).
- Action items assigned to "the team" or with no dates.
- Participants' names in a doc that reads as prosecution.
- No section asking where else the pattern exists.
- Last quarter's action items: status unknown, nobody's asked.

## Verification

- [ ] Timeline built from artifacts with detection gap and decision points — sources cited per row.
- [ ] Impact quantified (duration, users, cost, budget) — numbers in the doc.
- [ ] ≥3 contributing factors, none reducible to "person did X" without the system-level why.
- [ ] Every action item: owner, ticket, date, type (prevent/detect/mitigate) — table complete.
- [ ] Class-sweep done: the "where else?" section names checked locations — grep/audit linked.
- [ ] 30-day follow-through review scheduled with an owner — calendar/ticket linked.

## Example

Incident: 31-min checkout outage after a config push. First draft said "root cause: invalid config deployed." Factor analysis expanded it: (1) config format had no schema validation, (2) config bypassed canary (deploys canaried, configs didn't), (3) alert fired on symptom 9 min late because it watched server errors not conversion, (4) rollback took 12 min because config rollback was undocumented. Four action items, one per factor, each ticketed+owned; the class-sweep found two other services with uncanaried config paths. 30-day check: 4/4 done. The next bad config was caught by validation in CI — the postmortem's ROI in one line.

## Related skills

- `incident-triage` — the timeline log written during the incident is step 1's raw material.
- `alerting-design` — the "detect faster" action items.
- `runbook-authoring` — the "mitigate faster" action items.
