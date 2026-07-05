---
name: pricing-analysis
description: >
  Use when setting or restructuring prices — value-based vs cost-plus,
  tier and packaging design, price-metric choice, discount discipline,
  testing changes safely. Triggers: "what should we charge", "pricing
  tiers", "raise prices?", "discounting is out of control", "per-seat or
  usage pricing", "pricing page redesign". NOT for the unit-cost
  accounting (see unit-economics-model) or competitor teardowns (see
  competitive-analysis) — this turns both into a price.
---

# Pricing Analysis

## Overview

Cost floors, value ceilings, competition frames — and the price METRIC matters as much as the level, because a metric misaligned with delivered value generates the resentment that churns quietly. Behavioral willingness-to-pay evidence outranks stated, discounts are structure not exhortation, and pricing reviewed on a calendar beats pricing reviewed in a crisis.

## When to Use

- Setting initial pricing, restructuring tiers, or deciding a price change.
- Discount creep or plan-distribution weirdness signals the structure is leaking.

**When NOT to use:**
- Unit-cost accounting — `unit-economics-model` supplies the floor.
- Competitor mechanics — `competitive-analysis` supplies the landscape.

## Prerequisites

- Contribution margin per unit (the floor below which pricing is charity with invoicing).
- Segment-level willingness-to-pay evidence, or a plan to get it.

## The Workflow

1. **Anchor on value delivered, floor on cost, inform with competition:** the three inputs in their roles — cost sets the FLOOR (not the price: cost-plus leaves money on the table exactly when you're most valuable); competitor prices set the reference frame customers arrive carrying; value-to-customer sets the CEILING and target. Quantify value per segment where possible ("saves an ops manager 4h/week") — the defensible value story is also sales' best artifact.

2. **Get willingness-to-pay evidence beyond asking "would you pay?":** Van Westendorp's four questions for the acceptable RANGE per segment; conjoint when packaging trade-offs matter; and the strongest tier — behavior: win/loss-by-price from sales notes, discount depths that closed vs didn't, upgrade/downgrade flows. Stated intent inflates (`survey-design`'s rule); behavior decides.

3. **Choose the price metric as carefully as the level — it's the growth contract:** per-seat (simple, budgetable; taxes collaboration, invites seat-sharing), usage (scales with value; bill anxiety — pair with caps/commitments), flat/tiered (simplest; over/under-charges the extremes), hybrid (platform + usage — the B2B default for a reason). The test: does the metric track the value the customer receives, and grow when their value grows? Misalignment generates "we pay more for the thing we don't use more of" — the resentment that churns quietly.

4. **Design tiers around segment fences, not feature piles:** 3(±1) tiers mapped to real segments, each fenced by a feature that segment NEEDS and lower segments genuinely don't (SSO/audit/SLA fence enterprise cleanly; gating core value from entry just teaches the market you're not for them); the middle as the intended default; upgrade triggers occurring naturally in the product (the limit hit AT the moment of expansion — the fence the customer climbs willingly).

5. **Impose discount discipline as structure, not exhortation:** published guardrails (max % by size/term, approval tiers above); every discount gets something BACK (annual prepay, case study, multi-year — the free discount trains the market that list is fiction); the discount log analyzed quarterly by rep/segment — realized-vs-list widening is the structure announcing it's wrong somewhere.

6. **Test and roll out changes with a safety plan:** grandfather existing customers (or migrate with long notice + locked-rate window — the ambush converts a revenue lever into a churn event); new-customer cohorts take new pricing first (cleanest read: win-rate and realized price vs prior cohort); pre-commit the evaluation window and rollback criteria (`experiment-design`'s pre-registration — pricing has enough politics without post-hoc metric shopping).

7. **Review pricing on a calendar, not a crisis:** annual structural review (value delivered has grown — has price? the product that 3×'d capability at 2019 prices is donating the delta), quarterly health: realized-vs-list, plan distribution (everyone in one tier = the others mispriced or misfenced), expansion share, price-driven churn, win/loss-by-price. Pricing is the highest-leverage lever most companies touch least; the calendar fixes the touching-least part.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "Cost plus healthy margin — safe and defensible" | Margin-safe and value-blind: underpricing the segment that'd pay 3× and overpricing the one that won't. Cost is the floor's job; value sets the number. |
| "Customers told us $50 sounds fair" | The survey-stated $50 converts at $19. Behavioral evidence — what closed, what upgraded, what churned at which price — is the tier that decides (step 2's ranking). |
| "Per-seat is the industry standard — go with it" | Standard for products whose value scales with seats. Where value concentrates in two power users, per-seat invites seat-sharing — which is price-structure feedback, not customer dishonesty. |
| "The generous entry tier drives adoption" | The everything-tier is why nobody upgrades. Fences are built from what the NEXT segment needs (SSO, audit), not from rationing core value or from generosity vibes. |
| "One more discount closes the quarter" | Unstructured, get-nothing-back discounting compounds until realized price is 60% of list and every renewal anchors on the discounted number. The guardrails are quarterly revenue protection, not sales friction. |
| "Raising prices will trigger churn — don't touch it" | The example's fear was wrong: win-rate flat, realized/list +18 points. Untouched pricing on a 3×'d product is a silent donation — and grandfathering is precisely the churn-risk control. |

## Red Flags

- Prices unchanged for years while the product transformed.
- Realized-vs-list gap wide and widening, unmeasured.
- 85% of customers clustered in one tier.
- Discounts granted without reciprocity, logged nowhere.
- Seat-sharing rampant (metric misalignment presenting as abuse).
- A price change shipped to the installed base without grandfathering.

## Verification

- [ ] The three inputs documented: cost floor, value quantification per segment, competitive frame.
- [ ] WTP evidence includes the behavioral tier (win/loss-by-price, discount-depth outcomes) — sources attached.
- [ ] Metric choice justified against the value-tracking test — one paragraph.
- [ ] Fence audit done: each tier boundary named with the segment-need behind it.
- [ ] Discount guardrails published; reciprocity rule active; realized-vs-list dashboarded.
- [ ] Rollout plan: grandfathering terms, cohort-first sequencing, pre-registered evaluation + rollback criteria — doc linked.
- [ ] Annual review calendared; quarterly health metrics live.

## Example

B2B SaaS, symptoms: 85% of customers in the middle tier, realized price 71% of list, sales "everyone demands discounts," pricing untouched three years despite major product expansion. Evidence pass: Van Westendorp by segment (n=240) + 18 months of win/loss-by-price + discount logs. Findings: SMB's acceptable range sat BELOW the entry tier (explaining the discount pressure — list was fantasy for that segment); enterprise's range sat far above the top tier (the 85% clustering was enterprise buying mid-tier happily — money donated); and the per-seat metric fought the actual value driver (workflows run, concentrated in few seats — seat counts flat while usage 4×'d). Restructure: hybrid metric (platform fee + workflow bands), entry repriced into SMB's measured range, enterprise tier rebuilt behind SSO/audit/SLA fences at 2.4× the old top. Rollout: new cohorts first, 12-month grandfathering; pre-registered evaluation (2 quarters). Results: win-rate flat (the fear was wrong), realized/list 71% → 89%, expansion share doubled, price-driven churn 0.4% — and the first enterprise renewals at the new tier closed without discount.

## Related skills

- `unit-economics-model` — the margin floor and the retention curves pricing moves.
- `competitive-analysis` — the reference frame customers price against.
- `experiment-design` / `ab-test-analysis` — testing changes honestly.
- `survey-design` — WTP instruments that don't lead the witness.
