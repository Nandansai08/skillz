---
name: unit-economics-model
description: >
  Use when building or auditing per-unit profitability — CAC, LTV,
  contribution margin, payback period — with honest cost allocation and
  cohort discipline instead of pitch-deck arithmetic. Triggers: "what's
  our CAC", "LTV to CAC ratio", "unit economics for", "are we profitable
  per customer", "payback period", "contribution margin". NOT for
  whole-company cash planning (see cash-flow-forecast) or price-setting
  (see pricing-analysis, which consumes these margins).
---

# Unit Economics Model

## Overview

Pitch-deck arithmetic dies on four hills: marketing-only CAC, 1/churn LTV from one good month, revenue instead of margin, and the blended average hiding which segment subsidizes which. Honest unit economics are cohort-built, fully-loaded, segmented — with payback as co-headline, because payback is the number denominated in "when do we get our money back."

## When to Use

- Deciding whether growth spend is buying value or buying losses at scale.
- Auditing someone's LTV:CAC slide before it goes to a board or a bank.

**When NOT to use:**
- Whole-company cash planning — `cash-flow-forecast`.
- Setting prices — `pricing-analysis`.

## Prerequisites

- The unit chosen deliberately (customer, order, seat) — and held constant; mixing units mid-model is the genre's silent classic.
- Cohort-capable data (`cohort-retention-analysis`'s infrastructure).

## The Workflow

1. **Build contribution margin first — the floor everything stands on:** revenue per unit minus TRULY variable costs (COGS, payment processing, usage-scaling hosting, support per account, onboarding labor). The honesty test per cost: "if we add one customer, does this rise?" — commissions yes, the CFO no, AWS *partially* (decompose it). A "70% gross margin" ignoring support and payment fees is marketing; contribution is where "each sale helps" gets decided.

2. **Compute CAC fully-loaded and segmented:** ALL sales and marketing cost (salaries, tools, agencies — not just ad spend) ÷ new customers, lag-adjusted where cycles are long (this quarter's spend closes next quarter's customers). Then the mandatory splits: **blended vs paid** (organic customers in the denominator flatter paid channels) and per-channel/per-segment — the average across a $50 SMB channel and a $40k enterprise motion describes neither.

3. **Build LTV from cohort retention curves, not a churn snapshot:** LTV = contribution per period × expected lifetime, where lifetime comes from OBSERVED cohort decay (the plateau matters: a flattening curve supports long lifetimes; decay-to-zero doesn't, whatever the average says). Refuse the classic inflations: 1/churn on one good month ("2% monthly → 50-month lifetime" from 12 weeks of data); revenue-LTV instead of margin-LTV; discount-free far-future dollars (cap the horizon at 24–36 months — also all your data supports).

4. **Elevate payback period to co-headline — the cash truth LTV hides:** months of contribution to recover CAC. Two businesses at identical 4:1 LTV:CAC are utterly different at 6-month vs 30-month payback — the second finances years of CAC per cohort (`cash-flow-forecast`'s working-capital reality) and bets that retention holds past your data. Payback survives contact with a downturn.

5. **Segment before averaging — the aggregate lies by construction:** economics by size/channel/plan/vintage routinely reveal one segment subsidizing another (enterprise's 9-month payback funding SMB's never). The decision-grade artifact is the segment table; mix-shift over time (the average "improving" because the mix moved) is caught only here.

6. **Stress the load-bearing assumptions:** ±20% on retention-plateau height, CAC trend (channels saturate — early-adopter CAC is the cheapest you'll ever see), margin (support cost rises with product complexity). Which assumption flips the verdict? That's the number to instrument quarterly — the model's job is naming its own kill-switch.

7. **Wire it to decisions and recompute on cadence:** channel budgets by segment-level CAC-payback; the grow-vs-profitability debate conducted in payback months; quarterly recomputation against cohort actuals — the deck still citing the 2023 cohort's LTV is navigation by a dead star.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "CAC = ad spend / customers — the standard formula" | Minus the sales team, the tools, the agencies — understating CAC 2–3× in sales-led motions and flattering every downstream ratio. Fully-loaded or it's a different metric wearing CAC's name. |
| "Churn was 2% last month — LTV is 50 months of revenue" | One good month annualized into a five-year promise, from twelve weeks of data. Cohort curves with visible plateaus, or the horizon capped at what the data supports. |
| "5:1 LTV:CAC — the benchmark says we're great" | At 40% contribution margin that's 2:1 on money, and the first investor's analyst does the conversion for you, in the meeting. Margin-LTV or the ratio is theater. |
| "The blended number is stable — economics are fine" | While the paid channel's CAC doubled, hidden by organic growth in the denominator. Blends are where channel deterioration goes to hide (steps 2, 5). |
| "LTV:CAC is healthy — payback is a secondary detail" | The 'healthy' 4:1 at 30-month payback finances two-plus years of CAC per cohort into whatever the economy does next. Payback is the cash truth; the example's board decision turned on it. |
| "We computed all this for the fundraise last year" | Cohort actuals since then are checkable — and unchecked models drift into fiction precisely when growth spend scales. Quarterly recompute or the deck is a museum piece. |

## Red Flags

- CAC excluding sales salaries in a sales-led motion.
- LTV derived from 1/churn anywhere.
- Revenue-LTV:CAC presented without the margin conversion.
- One blended ratio, no segment table.
- Payback period absent from the headline.
- Model never reconciled against subsequent cohort behavior.

## Verification

- [ ] Contribution margin built with the per-cost honesty test — cost list attached.
- [ ] CAC fully-loaded, lag-adjusted, split blended/paid and per-segment — sources shown.
- [ ] LTV from cohort curves with plateau evidence; horizon within data support — curves attached.
- [ ] Payback co-reported with LTV:CAC — both in the headline.
- [ ] Segment table produced; mix-shift check done.
- [ ] Stress test run; the verdict-flipping assumption named and instrumented.
- [ ] Recompute cadence set; last actuals-reconciliation dated.

## Example

B2B SaaS, board asking "should we double paid acquisition?" — the deck said LTV:CAC 4.2:1, ship it. Audit: CAC was marketing-only (adding sales: 2.6:1); LTV was revenue-based on 1/churn from a strong quarter (rebuilt from 24 months of cohort curves at contribution margin — 68% gross became 54% contribution: 1.9:1). Payback: 19 months blended. Then the segment table changed the question: enterprise — payback 11 months, plateau 85%, genuinely strong; SMB paid-social — payback 31 months against a curve still decaying at month 18 (no plateau = the LTV was a hope). Stress test named the kill-switch: SMB flipped negative at −10% retention, inside observed variance. Decision shipped: double ENTERPRISE spend, cut SMB paid-social 60% pending a retention fix, recompute quarterly. Two quarters later the recompute showed the SMB plateau forming at month 20 post-onboarding-fix — spend partially restored, this time with the model watching.

## Related skills

- `cohort-retention-analysis` — the curves LTV must be built from.
- `pricing-analysis` — margins as pricing inputs.
- `cash-flow-forecast` — payback's company-level consequence.
- `financial-model-review` — the audit this model should survive.
