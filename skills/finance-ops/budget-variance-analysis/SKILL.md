---
name: budget-variance-analysis
description: >
  Use when explaining actual-vs-plan differences — decomposing variances
  into drivers, applying materiality thresholds, separating timing from
  real misses. Triggers: "explain the budget variance", "we're over/under
  budget", "actuals vs plan", "variance commentary", "why is spend 20%
  over". NOT for building the budget/forecast itself (see
  cash-flow-forecast for the cash view) — this interrogates the gap.
---

# Budget Variance Analysis

## Overview

Variance commentary earns its existence by adding cause, consequence, or action — "spend was higher due to increased costs" is the null explanation the genre must ban. Materiality gates focus the hours where the money is; nature-classification (timing/volume/rate/plan-error/one-time) decides everything downstream; and the basis check comes first because half of "variances" are bookkeeping artifacts.

## When to Use

- Monthly/quarterly close needs variance commentary that explains rather than restates.
- A line item blew through plan and leadership wants the why before the reforecast.

**When NOT to use:**
- Building the forward plan — `cash-flow-forecast` for cash; planning is upstream.

## Prerequisites

- Actuals and plan at matching granularity AND matching accounting basis (accrual actuals vs a cash-built plan manufactures fake variances — check FIRST, it's the most common silent error).
- The plan's assumptions (headcount timing, price, volume drivers).

## The Workflow

1. **Set materiality gates before looking, and analyze only what clears them:** e.g., investigate > ±5% AND > $10k (both — percentage alone flags a $200 blip; absolute alone buries a 40% miss on a scaling line). Everything below the gates gets one summary line. The gates keep the pack readable and the hours where the money is.

2. **Classify each material variance by its NATURE first — the triage that determines everything:**
   - **Timing:** real but shifted periods. Self-reversing — VERIFY it reverses next period; the "timing" claim three months old is a miss in a costume.
   - **Volume/mix:** more/less of the driver than planned.
   - **Rate/price:** same volume, different unit cost (cloud price change, FX).
   - **Plan error:** the assumption was wrong on day one (the forgotten renewal). Feeds next planning cycle, not a fire drill.
   - **One-time:** genuinely non-recurring — label it AND track the label's frequency per line (four consecutive "one-time" quarters IS the run rate).

3. **Decompose the big variances into driver math:** volume × rate splits wherever a driver exists — "cloud +$84k = +$61k usage (traffic +22%) + $23k rate (egress price change)" — because the components have different owners and different actions. Headcount lines: heads × timing × cost-per-head — which catches "under budget because hiring slipped" masquerading as discipline (underspend is frequently a delivery miss).

4. **Chase the material ones to their operational cause, not their accounting name:** "software over" is a category; the cause is "14 unused licenses auto-renewed." One conversation with the line's owner per material variance — the GL says where, the owner says why. Scan GROSS movements, not just nets: a line netting to zero because two errors cancelled is two problems reported as none.

5. **Write commentary as driver + action + go-forward, ≤3 lines per item:** "Cloud +$84k: 73% usage (traffic-driven, scales with revenue — reforecast raised), 27% egress rate (procurement engaging; resolution by Q3)." Every sentence adds cause, consequence, or action, or it's padding.

6. **Separate the reforecast implication from the explanation:** per material variance — does it change the full-year view? (Rate changes and structural volume shifts do; true timing and true one-times don't.) Roll implications into the reforecast explicitly rather than letting the annual number absorb surprises until Q4 detonates. The variance analysis IS the reforecast's evidence base — its actual job.

7. **Close the loop on recurring patterns:** the same line missing the same direction three periods running is a planning-assumption bug, not three coincidences; a variance log (item, driver, action, resolution) turns next quarter's analysis from archaeology into diffing.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "R&D was over plan due to higher R&D spend — commentary done" | The null explanation: the number restated in words. Driver math or silence; leadership can read the number themselves. |
| "It's timing — it'll reverse" | Then name the reversal period and check it. 'Timing' claimed in March and unexplained in June was a miss wearing the label — the eternal-timing dodge is the genre's oldest trick. |
| "Under budget — good news, move on" | $200k under on engineering = three unfilled roles = the roadmap slipping. The headcount decomposition (step 3) is what distinguishes savings from a delivery miss celebrated by accident. |
| "The department nets to zero — on budget overall" | A big overrun and a big underrun cancelling is two problems reported as none (the example's contractor/salary pair). Gross movements or the net conceals. |
| "Commentary on every line shows thoroughness" | Forty slides nobody reads, with the material item buried on slide 23. The gates exist so attention lands where the money is. |
| "That's one-time — exclude it from the run rate" | Fourth consecutive quarter of 'one-time' items IS the run rate laundered. Track the label's frequency per line; the pattern is the finding. |

## Red Flags

- Commentary that restates numbers in words.
- Basis mismatch never checked; hours spent investigating bookkeeping artifacts.
- "Timing" entries with no reversal date, recurring across periods.
- Netted lines hiding offsetting errors.
- No owner conversations — causes guessed from the GL.
- The same line missing the same way for three periods, assumption unchanged.

## Verification

- [ ] Basis match confirmed before any analysis — noted.
- [ ] Materiality gates stated; below-gate items summarized in one line.
- [ ] Every material variance nature-classified; timing claims carry reversal dates (and prior claims verified reversed).
- [ ] Driver math shown for the big items (volume × rate, heads × timing × cost).
- [ ] Operational cause per material item from the owner conversation — named.
- [ ] Reforecast implications separated and rolled — the full-year view updated.
- [ ] Variance log current — link.

## Example

Quarterly close: engineering +$210k over plan (8%), CFO wants the story. Gates cleared by three lines. Basis check first: clean. Cloud +$84k — decomposed (73% usage tracking traffic growth, 27% an egress rate change; two owners, two actions, reforecast raised for the usage component). Contractors +$150k — owner conversation revealed the cause: two backfills running as contractors because hiring slipped (the offsetting entry: salaries −$95k "favorable") — reported as ONE story ("hiring miss costing a net $55k premium plus delivery risk"), defusing the netting trap. Software +$31k — unused-license auto-renewal, a plan error (renewal calendar now feeds planning; licenses reclaimed — $18k/quarter recovered). Commentary: nine lines total. Next quarter the variance log's payoff: cloud's usage line was the only repeat — and its driver model had been fixed, so the commentary was one line: "tracking the revised model, ±2%."

## Related skills

- `cash-flow-forecast` — the forward-looking sibling this analysis calibrates.
- `kpi-reporting-pack` — the reporting vehicle the commentary rides in.
- `metric-definition` — driver models with honest numerators and denominators.
- `financial-model-review` — auditing the plan model these variances expose.
