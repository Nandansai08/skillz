---
name: budget-variance-analysis
description: >
  Use when explaining actual-vs-plan differences — decomposing variances
  into drivers, applying materiality thresholds, separating timing from
  real misses. Triggers: "explain the budget variance", "we're over/under
  budget", "actuals vs plan", "variance commentary", "why is spend 20% over".
---

# Budget Variance Analysis

## When to use this skill
- Monthly/quarterly close needs variance commentary that explains rather than restates.
- A line item blew through plan and leadership wants the why before the reforecast.
- NOT for building the budget/forecast itself (`cash-flow-forecast` for the cash view) — this skill interrogates the gap between a plan that exists and what happened.

## Prerequisites
- Actuals and plan at matching granularity and matching accounting basis (comparing accrual actuals to a cash-built plan manufactures fake variances — check this FIRST, it's the most common silent error).
- The plan's assumptions (headcount timing, price, volume drivers) — variance analysis without the plan's logic is arithmetic without meaning.

## Workflow

1. **Set materiality gates before looking, and analyze only what clears them:** e.g., investigate variances > ±5% AND > $10k (both conditions — percentage alone flags a $200 blip on a tiny line; absolute alone buries a 40% miss on a small-but-scaling line). Everything below the gates gets one summary line. The gates are what keep the pack readable and the analysis hours where the money is; commentary on every line is how variance decks become 40 slides nobody reads (`kpi-reporting-pack`'s same discipline).

2. **Classify each material variance by its NATURE first — the triage that determines everything downstream:**
   - **Timing:** the expense/revenue is real but shifted periods (invoice landed early, project slipped a month). Self-reversing — verify it actually reverses next period rather than re-explaining it monthly forever (the "timing" that's three months old is a miss wearing a timing costume).
   - **Volume/mix:** more/less of the driver than planned (headcount, usage, deals).
   - **Rate/price:** same volume, different unit cost (cloud price change, comp adjustments, FX).
   - **Plan error:** the assumption was wrong on day one (the contract renewal everyone forgot). Feeds the next planning cycle, not a fire drill.
   - **One-time:** genuinely non-recurring (legal settlement, migration cost). Label it AND track the label's abuse (see pitfalls).

3. **Decompose the big variances into driver math:** volume × rate splits wherever the line has a driver — "cloud spend +$84k = +$61k usage growth (traffic +22%, roughly proportional) + $23k rate (egress price change)" — because the two components have different owners and different actions (usage → engineering efficiency conversation; rate → procurement/`vendor-evaluation` conversation). For headcount-driven lines: heads × timing × cost-per-head decomposition catches the "under budget because hiring slipped" masquerading as discipline (`capacity-planning`'s forecast-miss cousin — underspend is frequently a delivery miss, not a savings).

4. **Chase the material ones to their operational cause, not their accounting name:** "software spend over" is a category, not a cause — the cause is "14 unused licenses auto-renewed" or "the team bought the tool procurement was still negotiating." One conversation with the line's owner per material variance; the GL tells you where, the owner tells you why (`production-debugging`'s what-changed instinct, applied to money). Watch for the offsetting-error trap: a line that nets to zero because two large errors cancelled is two problems reported as none — scan gross movements, not just nets.

5. **Write commentary as driver + action + go-forward, in three lines or fewer per item:** "Cloud +$84k: 73% usage (traffic-driven, scales with revenue — reforecast raised accordingly), 27% egress rate change (procurement engaging vendor; expect resolution by Q3). No action needed on usage component." The banned genre: commentary that restates the number in words ("spend was higher than planned due to increased costs") — every sentence must add cause, consequence, or action, or it's padding (`natural-prose-editing`'s concrete-over-vague, financial edition).

6. **Separate the reforecast implication from the explanation:** for each material variance — does it change the full-year view? (rate changes and structural volume shifts do; true timing and true one-times don't); roll the implications into the reforecast/outlook explicitly rather than letting the annual number quietly absorb surprises until Q4 detonates. The variance analysis is the reforecast's evidence base — that's its actual job, beyond the commentary ritual.

7. **Close the loop on the recurring patterns:** the same line missing the same direction three periods running is a planning-assumption bug, not three coincidences (fix the assumption — and the `metric-definition`-grade driver model behind it); "one-time" items appearing every quarter are a category error institutionalized; and a variance log (item, driver, action, resolution) turns next quarter's analysis from archaeology into diffing (`on-call-handoff`'s trend-across-notes payoff, finance edition).

## Common pitfalls
- Restatement commentary: "R&D was $40k over plan due to higher R&D spend" — the null explanation, and the most common sentence in variance decks. Driver math (step 3) or silence.
- The eternal timing variance: "timing" claimed in March, still unexplained in June — timing claims come with a reversal date, and the date gets checked (step 2's verify).
- Basis mismatch fake variances: accrual actuals vs cash plan, or plan-FX vs actual-FX — hours of investigating differences that are bookkeeping artifacts. The prerequisite check, always first.
- One-time laundering: every overrun labeled non-recurring so the run-rate looks clean — four consecutive "one-time" quarters IS the run rate. Track the label's frequency per line.
- Celebrating underspend uncritically: $200k under on engineering = three unfilled roles = the roadmap slipping — the favorable variance that's an unfavorable outcome (step 3's headcount decomposition).
- Netting concealment: department "on budget" because a big overrun and a big underrun cancelled — gross movements scanned, offsets named (step 4's trap).

## Example
Quarterly close: engineering department +$210k over plan (8%), CFO wants the story. Gates cleared by three lines. Basis check first: clean. Cloud +$84k — decomposed per step 3 (73% usage tracking traffic growth, 27% an egress rate change; two owners, two actions, reforecast raised for the usage component). Contractors +$150k — owner conversation revealed the cause: two backfills running as contractors because hiring slipped (the offsetting entry: salaries −$95k "favorable") — reported as ONE story ("hiring miss costing a net $55k premium plus delivery risk"), not two variances, defusing the netting trap. Software +$31k — the unused-license auto-renewal, a plan error (renewal calendar now feeds planning; licenses reclaimed — $18k/quarter recovered go-forward). Commentary: nine lines total for the three items. Reforecast implications separated: cloud usage structural (raised), contractor premium temporary-with-date (hiring pipeline attached), software resolved. The variance log's payoff arrived next quarter: cloud's usage line was the only repeat — and its driver model (spend per traffic unit) had been fixed, so the commentary was one line: "tracking the revised model, ±2%."

## Related skills
- `cash-flow-forecast` — the forward-looking sibling this analysis calibrates.
- `kpi-reporting-pack` — the reporting vehicle the commentary rides in.
- `metric-definition` — driver models with honest numerators and denominators.
- `financial-model-review` — auditing the plan model these variances expose.
