---
name: unit-economics-model
description: >
  Use when building or auditing per-unit profitability — CAC, LTV,
  contribution margin, payback period — with honest cost allocation and
  cohort discipline instead of pitch-deck arithmetic. Triggers: "what's our
  CAC", "LTV to CAC ratio", "unit economics for", "are we profitable per
  customer", "payback period", "contribution margin".
---

# Unit Economics Model

## When to use this skill
- Deciding whether growth spend is buying value or buying losses at scale.
- Auditing someone's LTV:CAC slide before it goes to a board or a bank.
- NOT for whole-company cash planning (`cash-flow-forecast`) or price-setting itself (`pricing-analysis` — which consumes these margins as inputs).

## Prerequisites
- The unit chosen deliberately: customer, account, order, seat, transaction — the right unit is the one decisions are made about, and mixing units mid-model (CAC per customer, margin per order) is the genre's silent classic.
- Cohort-capable data: revenue and costs attributable by customer/period (`cohort-retention-analysis`'s infrastructure).

## Workflow

1. **Build contribution margin first — the floor everything stands on:** revenue per unit minus TRULY variable costs (COGS, payment processing, hosting-that-scales-with-usage, support cost per account, onboarding labor). The honesty test per cost: "if we add one more customer, does this cost rise?" — sales commissions yes, the CFO's salary no, the AWS bill *partially* (decompose it: the per-tenant compute yes, the baseline cluster no). A "70% gross margin" that ignores support and payment fees is marketing; the contribution line is where "each sale helps" or "we lose money on every unit and make it up in volume" gets decided.

2. **Compute CAC fully-loaded and segmented:** all sales AND marketing cost (salaries, tools, agencies, content — not just ad spend) ÷ new customers acquired, matched in period with a lag adjustment where cycles are long (this quarter's spend closes next quarter's customers — divide spend by the customers it actually produced, not the calendar's coincidence). Then the mandatory split: **blended vs paid CAC** (organic customers in the denominator flatter paid channels' true cost) and per-channel/per-segment (the average CAC across a $50 SMB channel and a $40k enterprise motion describes neither — decisions happen per channel).

3. **Build LTV from cohort retention curves, not from a churn snapshot:** LTV = contribution margin per period × expected lifetime, where lifetime comes from OBSERVED cohort decay (`cohort-retention-analysis`'s curves — the plateau matters: a curve that flattens supports long lifetimes; one decaying to zero doesn't, whatever the average says). The classic inflations to refuse: 1/churn arithmetic on one good month's churn rate (a 2% month → "50-month lifetime" extrapolated from 12 weeks of data); revenue-LTV instead of margin-LTV (a 3:1 revenue-LTV:CAC at 40% contribution margin is barely above water); and discount-free far-future dollars (money in year 5 isn't money today — discount it, or cap the horizon honestly at 24–36 months, which is also all your data supports).

4. **Elevate payback period to co-headline — it's the cash truth LTV hides:** months of contribution margin to recover CAC. Two businesses with identical 4:1 LTV:CAC are utterly different at 6-month vs 30-month payback — the second needs years of financing per cohort (`cash-flow-forecast`'s working-capital reality), and its model is a bet that retention holds for years past your data. Benchmarks are context-dependent, but payback is the number that survives contact with a downturn, because it's the one denominated in "when do we get our money back."

5. **Segment before averaging — the aggregate lies by construction:** unit economics by segment (size, channel, plan, geography, cohort-vintage) routinely reveals one segment subsidizing another (enterprise's 9-month payback funding SMB's never; the channel whose CAC doubled hiding inside a stable blend). The decision-grade artifact is the segment table, not the company average — and mix-shift over time (average "improving" because the mix moved, not because anything improved) is caught only here (`metric-definition` step 5's ratio caution, compounded).

6. **Stress the model's load-bearing assumptions:** ±20% on retention-plateau height, CAC trend (channels saturate — early-adopter CAC is the cheapest you will ever see), and margin (support cost per account tends to rise with product complexity). Which assumption flips the verdict? That's the number to instrument and re-check quarterly — the model's job is naming its own kill-switch, not producing one blessed ratio (`financial-model-review` will hunt for exactly this when auditing you).

7. **Wire it to decisions and recompute on cadence:** channel budgets reallocated by segment-level CAC-payback (not blended LTV:CAC); the "grow vs profitability" debate conducted in payback months; pricing changes fed from the margin decomposition (`pricing-analysis`); and quarterly recomputation against actuals — unit economics are a moving measurement of a moving business, and the deck still citing the 2023 cohort's LTV is navigation by an old star chart.

## Common pitfalls
- Marketing-only CAC: ad spend ÷ customers, sales team's cost vanished — understates CAC 2–3× in sales-led motions and flatters every downstream ratio.
- The 1/churn LTV on thin data: one good month annualized into a 50-month lifetime, multiplied into a valuation. Cohort curves or humility (step 3).
- Revenue-LTV:CAC presented without the margin conversion — the 5:1 that's actually 1.8:1 after contribution margin, discovered by the first investor who asks.
- Blend-hiding: the paid channel's CAC doubling quietly inside a blended number stabilized by organic growth — per-channel or it's decoration (steps 2, 5).
- Ignoring payback: the "great" 4:1 business that finances 30 months of CAC per cohort into a rising-rate environment (step 4's cash truth).
- Static economics: computed once for the fundraise, never reconciled against subsequent cohort actuals — the model's predictions are checkable within quarters; unchecked models drift into fiction (step 7).

## Example
B2B SaaS, board asking "should we double paid acquisition?" — the deck said LTV:CAC 4.2:1, ship it. Audit first: CAC was marketing-only (adding sales: 2.6:1); LTV was revenue-based on 1/churn from a strong quarter (rebuilt from 24 months of cohort curves at contribution margin — 68% gross became 54% contribution after support and payment costs: 1.9:1). Payback: 19 months blended. Then the segment table changed the question entirely: enterprise (sales-led) — payback 11 months, retention plateau 85%, economics genuinely strong; SMB paid-social — payback 31 months against a cohort curve still decaying at month 18 (no plateau = the LTV was a hope). Stress test named the kill-switch: SMB economics flipped negative at −10% retention, well inside observed variance. Decision that shipped: double ENTERPRISE acquisition spend, cut SMB paid-social 60% pending a retention fix (the onboarding work from the `cohort-retention-analysis` example became its prerequisite), recompute quarterly against cohort actuals. Two quarters later the recompute showed SMB plateau forming at month 20 post-onboarding-fix — spend partially restored, this time with the model watching.

## Related skills
- `cohort-retention-analysis` — the curves LTV must be built from.
- `pricing-analysis` — margins as pricing inputs.
- `cash-flow-forecast` — payback's company-level consequence.
- `financial-model-review` — the audit this model should survive.
