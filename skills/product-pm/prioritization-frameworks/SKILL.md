---
name: prioritization-frameworks
description: >
  Use when ranking a backlog or roadmap candidates — applying RICE/ICE/Kano
  appropriately, avoiding scoring theater, making the trade-offs legible.
  Triggers: "prioritize this backlog", "RICE scoring", "what should we build
  next", "too many priorities", "rank these features", "everything is P0".
---

# Prioritization Frameworks

## When to use this skill
- A pile of candidate work exceeds capacity and the ordering is contested.
- An existing scoring process has become theater (scores computed, decisions made elsewhere).
- NOT for sprint-level task ordering (that's mostly dependency + risk sequencing, `user-story-slicing` step 6) — this is the quarter-scale "which bets" question.

## Prerequisites
- A strategy or goal statement to prioritize *against* — frameworks rank items toward an objective; with no objective, scoring is numerology. If the strategy is missing, that's the first conversation, not this one.

## Workflow

1. **Pick the framework by decision type, not fashion:**
   - **RICE** (Reach × Impact × Confidence ÷ Effort): comparing many candidate features where reach genuinely differs and some data exists. Its distinctive virtue is the Confidence term — it forces "how do we know?"
   - **ICE**: RICE for early stages where reach estimates would be fiction anyway. Faster, coarser, fine.
   - **Kano** (basic / performance / delighter classification via user reactions): deciding *what kind* of investment a feature is — it answers "will this differentiate or is it table stakes?", not "what's first?" Use it to balance the portfolio, then RICE within.
   - **Weighted shortest job first** (cost-of-delay ÷ duration): when timing dominates — deadlines, decaying opportunities, compliance dates.
   - None of the above for: strategic bets and platform investments (step 5).

2. **Score with rules that resist gaming:** define the scales *before* scoring (Impact: 3 = moves the target metric ≥5%, 1 = measurable, 0.5 = plausible — tied to `metric-definition`-grade metrics, not vibes); Reach from actual data (funnel counts, ticket volumes), with the source cited per score; Confidence as evidence-tier (1.0 = experiment/data, 0.8 = strong user research, 0.5 = informed opinion, 0.2 = someone senior said so — writing "0.2" next to the pet feature does more than any argument); Effort from the people who'd build it, in coarse units (person-weeks, capped precision — false precision in effort is the #1 score corrupter).

3. **Use scores to structure argument, not to end it.** The output of scoring is the *sorted list plus the surprises*: items whose score-rank contradicts gut-rank get discussed — sometimes the score is wrong (a Reach estimate was garbage), sometimes the gut is (recency bias, loudest-customer bias). The framework's real product is that this discussion happens over named assumptions instead of adjectives. Decision rights stay human; anyone "just shipping the spreadsheet order" has automated judgment, not exercised it.

4. **Force ranking, forbid tiers of everything:** the deliverable is an ordered list where position N+1 starts only if capacity remains — not P0/P1/P2 buckets where P0 holds 60% of items ("everything is P0" is the null prioritization). The forcing question for ties: "if we could do only one, which?" — repeat until ordered.

5. **Handle the three framework-breaking categories explicitly, outside the scores:** platform/debt work (its Reach is other teams' velocity — score it by what it *unblocks*, or reserve a capacity fraction: the 15–20% infrastructure allocation beats making debt items lose to features forever); strategic bets (low Confidence *by definition* — a portfolio decision: "one bet per quarter", not a RICE loser); and compliance/contractual deadlines (they're constraints, not candidates — they come off the top before ranking).

6. **Timebox the whole exercise and publish the result with reasons:** scoring 40 items to two decimals over three weeks costs more than the misordering it prevents — a half-day with the right people, coarse scores, then step 3's discussion. Publish: the ordered list, the top items' one-line why, and *what didn't make it and why* (`stakeholder-update` and `roadmap-communication` carry this outward — the "no" list with reasons is what makes the next request cycle saner).

7. **Re-rank on a cadence tied to information, not on every stakeholder ping:** quarterly, or when material new evidence lands (a competitor move, a churn analysis, a failed bet). Between cycles, new items enter a triage lane against the *bottom* of the funded list, not against a panic re-litigation of the top. Log score-vs-outcome for shipped items once or twice a year — Reach and Effort estimates have systematic biases (usually 2× optimistic), and the log is what calibrates the next cycle.

## Common pitfalls
- Scoring theater: elaborate RICE spreadsheet, decisions still made by the loudest voice — worse than no framework, because it launders the outcome. Step 3's surprises-discussion is the difference.
- Effort in the denominator with false precision: someone's "3 weeks" vs "8 weeks" guess swings rank more than any Impact debate; coarse units + builder-sourced estimates contain it.
- Confidence theater in reverse — everything scored 0.8 because writing 0.3 feels rude. Tie the tiers to evidence types (step 2) so the number is a fact-claim, not a mood.
- Kano misused as a ranker ("delighters first!") — it classifies investment types; a product of all delighters with broken basics churns.
- Cross-team score comparisons ("their 42 beats our 38") — scales calibrate within a scoring group only; comparing across groups compares calibrations.
- The unranked graveyard: items #23–60 kept "on the backlog" forever, absorbing grooming time. Below the fund line + one cycle untouched = close with a reason (searchable), reopen if evidence returns.

## Example
Quarter planning, 31 candidates, goal: reduce SMB churn (target metric defined per `metric-definition`). Compliance item (tax-form change, hard date) pulled off the top as a constraint; 15% capacity reserved for platform (the CI-speed work that would never win a churn-framed RICE but was throttling everything — the step-5 reservation). Remaining 27 RICE-scored in a half-day: Reach from support-ticket and funnel counts (cited), Impact tiers tied to the churn metric, Confidence by evidence tier — which produced the exercise's key moment: the CEO's favorite (marketplace integrations) scored Confidence 0.2 (zero research) against the boring payout-reconciliation fix at 0.8 (23% of churn calls, 6 interviews — the `prd-writing` example's evidence, upstream). Step-3 discussion: rather than kill the favorite, it got a 2-week research spike (cheap Confidence-raiser) while reconciliation took the top build slot. Published list included the "not this quarter" section with reasons. Retro two quarters later: the calibration log showed Reach estimates ran 1.8× optimistic — applied as a haircut in the next cycle's scoring.

## Related skills
- `metric-definition` — the Impact scale's foundation.
- `roadmap-communication` — publishing the output without promising dates.
- `feature-scoping-cut` — shrinking the winner to fit its slot.
- `capacity-planning` — the engineering-capacity reality behind the fund line.
