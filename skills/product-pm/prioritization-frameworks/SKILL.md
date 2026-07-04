---
name: prioritization-frameworks
description: >
  Use when ranking a backlog or roadmap candidates — applying RICE/ICE/
  Kano appropriately, avoiding scoring theater, making the trade-offs
  legible. Triggers: "prioritize this backlog", "RICE scoring", "what
  should we build next", "too many priorities", "rank these features",
  "everything is P0". NOT for sprint-level task ordering (dependency +
  risk sequencing) — this is the quarter-scale "which bets" question.
---

# Prioritization Frameworks

## Overview

Frameworks structure the argument; they don't end it. Scores exist to surface named assumptions and rank-gut disagreements for discussion — the deliverable is a force-ranked list with reasons, and anyone "just shipping the spreadsheet order" has automated judgment rather than exercised it.

## When to Use

- Candidate work exceeds capacity and the ordering is contested.
- An existing scoring process has become theater (scores computed, decisions made elsewhere).

**When NOT to use:**
- Sprint-level task ordering — dependency + risk sequencing (`user-story-slicing` step 6).

## Prerequisites

- A strategy or goal to prioritize *against* — with no objective, scoring is numerology. If the strategy is missing, that's the first conversation, not this one.

## The Workflow

1. **Pick the framework by decision type, not fashion:**
   - **RICE** (Reach × Impact × Confidence ÷ Effort): many candidates, reach genuinely differs, some data exists. Its virtue is the Confidence term — it forces "how do we know?"
   - **ICE:** RICE for early stages where reach estimates would be fiction.
   - **Kano** (basic/performance/delighter): answers "will this differentiate or is it table stakes?" — a portfolio balancer, NOT a ranker.
   - **Weighted shortest job first** (cost-of-delay ÷ duration): when timing dominates — deadlines, decaying opportunities.
   - None of the above for strategic bets and platform work (step 5).

2. **Score with rules that resist gaming:** scales defined *before* scoring (Impact: 3 = moves the target metric ≥5%, tied to `metric-definition`-grade metrics); Reach from actual data, source cited per score; Confidence as evidence-tier (1.0 = experiment, 0.8 = strong research, 0.5 = informed opinion, 0.2 = someone senior said so — writing "0.2" next to the pet feature does more than any argument); Effort from the people who'd build it, in coarse units (false precision in effort is the #1 score corrupter).

3. **Use scores to structure argument, not to end it.** The output is the *sorted list plus the surprises*: items whose score-rank contradicts gut-rank get discussed — sometimes the score is wrong (garbage Reach estimate), sometimes the gut is (recency bias, loudest-customer bias). The framework's real product is that this discussion happens over named assumptions instead of adjectives.

4. **Force ranking, forbid tiers of everything:** the deliverable is an ordered list where position N+1 starts only if capacity remains — not P0/P1/P2 buckets where P0 holds 60% of items. The forcing question for ties: "if we could do only one, which?"

5. **Handle the three framework-breaking categories explicitly, outside the scores:** platform/debt work (its Reach is other teams' velocity — score by what it *unblocks*, or reserve a capacity fraction: 15–20% beats making debt items lose to features forever); strategic bets (low Confidence *by definition* — a portfolio decision: "one bet per quarter"); compliance/contractual deadlines (constraints, not candidates — off the top before ranking).

6. **Timebox the whole exercise and publish the result with reasons:** a half-day with the right people, coarse scores, then step 3's discussion. Publish: the ordered list, the top items' one-line whys, and *what didn't make it and why* — the "no" list with reasons is what makes the next request cycle saner.

7. **Re-rank on a cadence tied to information, not on every stakeholder ping:** quarterly, or on material new evidence. Between cycles, new items triage against the *bottom* of the funded list, not a panic re-litigation of the top. Log score-vs-outcome for shipped items yearly — Reach and Effort estimates have systematic biases (usually ~2× optimistic), and the log calibrates the next cycle.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "The spreadsheet ranked it #1 — decision made" | Scores structure the argument; shipping the spreadsheet order launders whatever garbage estimate drove it. The surprises-discussion (step 3) is where judgment actually happens. |
| "Everything in P0 is genuinely critical" | P0 holding 60% of items is the null prioritization — it delegates the real ordering to whoever grabs tickets first. Force-rank or admit nobody decided. |
| "Confidence 0.8 across the board — the team believes in the work" | Uniform confidence is politeness, not evidence. Tie tiers to evidence types so 0.2 is a fact-claim about research, not an insult — that's what makes the pet feature's number honest. |
| "Effort estimates to the half-week — precision matters" | The denominator's false precision swings rank more than any Impact debate, and the guesses are ±2× anyway. Coarse units from the builders contain the damage. |
| "Kano says delighters win — build the delightful stuff" | Kano classifies investment types; a product of delighters with broken basics churns. It balances the portfolio; RICE ranks within it. |
| "Big customer asked, so it jumps the queue" | The between-cycles triage lane exists so renewals don't reorder the quarter weekly. If the evidence is material, re-rank; if it's volume, the lane holds. |

## Red Flags

- Scores computed after the decision was already made.
- Items #23–60 groomed quarterly, funded never, closed never.
- Effort estimates from people who won't build it.
- Confidence column all 0.8s.
- The published list with no "didn't make it" section.
- Cross-team score comparisons settling budget arguments.

## Verification

- [ ] Scales + evidence-tiers defined before scoring — doc predates the scores.
- [ ] Every Reach figure carries its data source — spot-check three.
- [ ] Surprises discussed: rank-vs-gut mismatches listed with resolutions — meeting notes.
- [ ] Output force-ranked; constraints and reserved capacity handled off-list — published artifact.
- [ ] "Not this quarter" section with reasons — published.
- [ ] Calibration log updated for last cycle's shipped items — biases noted and applied.

## Example

Quarter planning, 31 candidates, goal: reduce SMB churn. Compliance item (tax-form change, hard date) pulled off the top as a constraint; 15% capacity reserved for platform (the CI-speed work that would never win a churn-framed RICE but was throttling everything). Remaining 27 RICE-scored in a half-day: Reach from ticket and funnel counts (cited), Impact tiers tied to the churn metric, Confidence by evidence tier — which produced the exercise's key moment: the CEO's favorite (marketplace integrations) scored Confidence 0.2 (zero research) against the boring payout-reconciliation fix at 0.8 (23% of churn calls, 6 interviews). Step-3 discussion: rather than kill the favorite, it got a 2-week research spike (cheap Confidence-raiser) while reconciliation took the top build slot. Published list included the "not this quarter" section with reasons. Retro two quarters later: the calibration log showed Reach estimates ran 1.8× optimistic — applied as a haircut in the next cycle.

## Related skills

- `metric-definition` — the Impact scale's foundation.
- `roadmap-communication` — publishing the output without promising dates.
- `feature-scoping-cut` — shrinking the winner to fit its slot.
- `capacity-planning` — the engineering-capacity reality behind the fund line.
