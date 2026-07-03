---
name: blameless-postmortem
description: >
  Use after an incident is resolved to write the postmortem — timeline,
  contributing factors, action items that actually get done, without blame.
  Triggers: "write the postmortem", "incident review", "RCA document",
  "post-incident review", "lessons learned from the outage".
---

# Blameless Postmortem

## When to use this skill
- After any SEV1/SEV2, near-misses included (near-misses are free postmortems).
- Reviewing someone else's postmortem draft.
- NOT during the incident — that's `incident-triage`; this starts once users are restored.

## Prerequisites
- The incident timeline log (channel history, deploy records, alert timestamps).
- Scheduled within 48h of resolution — memories of "why did we think that?" decay fast.
- A template — see [templates/postmortem.md](templates/postmortem.md).

## Workflow

1. **Reconstruct the timeline from artifacts, not memory.** Timestamps from: alerts, deploys, chat messages, dashboard annotations, commands run. Include the *detection gap* (impact start → first human awareness) and *decision points* ("14:20: considered rollback, chose fix-forward because X") — the decisions are where the learning lives.

2. **Write impact in numbers:** duration, users/requests affected, revenue/SLA/error-budget cost, support tickets. "Bad outage" is not analyzable; "31 min, 12% of checkouts failed, ~$40k GMV, 60% of monthly error budget" is.

3. **Hunt contributing factors, not THE root cause.** Real incidents are 3–6 factors aligning. Use "why did the system allow this?" not "who did this?" — every time the answer is a person's action, ask what made that action reasonable and undetectable:
   - ~~"Dev pushed a bad config"~~ → config had no validation, no canary, and review couldn't see the generated output.
   Categories to probe: the trigger, why defenses missed it (tests, review, canary), why detection was slow, why mitigation was slow, what amplified it.

4. **Enforce blamelessness structurally.** Names replaced by roles in the doc. The test: could every participant hand this doc to their manager without flinching? Blame isn't kindness policy — it's data quality: the moment postmortems punish, people stop reporting near-misses and start editing timelines, and you lose the sensor network.

5. **Write action items that survive contact with the backlog.** Each one: specific, owned by a name, ticketed, deadlined, and sized honestly. Split into:
   - **Prevent recurrence** (fix the class, not the instance — validation, guardrail, removal of the trap);
   - **Detect faster** (the alert that was missing — see `alerting-design`);
   - **Mitigate faster** (runbook, kill switch, drill).
   Three excellent items beat twelve aspirational ones; "improve testing culture" is not an action item.

6. **Extract the transferable lesson.** Ask: where *else* does this pattern exist? (Same config system, same missing canary, sibling services.) The instance is fixed; the class is the payoff — one grep or audit action item usually covers it (`regression-test-from-bug` step 6 at system scale).

7. **Review the follow-through, not just the doc.** 30 days later: are the action items done? Postmortem programs die not from bad docs but from action items rotting — track completion rate as a team metric, and re-raise stale items in the next review.

## Common pitfalls
- "Root cause: human error." Banned phrase. Humans err at a constant rate; systems determine whether errors become outages. If human error explains it, the analysis stopped one "why" too early.
- The counterfactual pile-on ("we should have known/tested/checked") without asking why the knowledge wasn't available *at the time*. Hindsight is the analysis's main contaminant.
- Timeline that starts at detection. The interesting part is often before: the change that armed the trap days earlier, the alert that fired and was dismissed.
- Action items assigned to "the team" — that's assigned to nobody.
- Postmortem theater: beautiful doc, no completed items, same incident in Q3. Step 7 is the difference between a ritual and a practice.
- Skipping postmortems for near-misses — they're identical learning at zero user cost.

## Example
Incident: 31-min checkout outage after a config push. First draft said "root cause: invalid config deployed." Factor analysis expanded it: (1) config format had no schema validation, (2) config bypassed canary (deploys canaried, configs didn't), (3) alert fired on symptom 9 min late because it watched server errors not conversion, (4) rollback took 12 min because config rollback was undocumented. Four action items, one per factor, each ticketed+owned; the class-sweep (step 6) found two other services with uncanaried config paths. 30-day check: 4/4 done. The next bad config was caught by validation in CI — the postmortem's ROI in one line.

## Related skills
- `incident-triage` — the timeline log written during the incident is step 1's raw material.
- `alerting-design` — the "detect faster" action items.
- `runbook-authoring` — the "mitigate faster" action items.
