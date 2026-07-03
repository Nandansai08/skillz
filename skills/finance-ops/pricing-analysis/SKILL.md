---
name: pricing-analysis
description: >
  Use when setting or restructuring prices — value-based vs cost-plus,
  tier and packaging design, price-metric choice, discount discipline,
  testing changes safely. Triggers: "what should we charge", "pricing
  tiers", "raise prices?", "discounting is out of control", "per-seat or
  usage pricing", "pricing page redesign".
---

# Pricing Analysis

## When to use this skill
- Setting initial pricing, restructuring tiers, or deciding a price change.
- Discount creep or plan-distribution weirdness signals the structure is leaking.
- NOT for the unit-cost accounting itself (`unit-economics-model` supplies the floor) or competitor teardown mechanics (`competitive-analysis` supplies the landscape) — this skill turns both into a price.

## Prerequisites
- Contribution margin per unit (`unit-economics-model` step 1) — the floor below which pricing is charity with invoicing.
- Segment-level willingness-to-pay evidence, or a plan to get it (step 2) — pricing without it is guessing with confidence.

## Workflow

1. **Anchor on value delivered, floor on cost, inform with competition:** the three inputs in their correct roles — cost sets the FLOOR (not the price: cost-plus leaves money on the table exactly when you're most valuable, and drowns you when you're not); competitor prices set the reference frame customers arrive carrying (`competitive-analysis` step 3's strategy-reading); value-to-customer sets the CEILING and the target. Quantify value per segment where possible ("saves an ops manager 4h/week" — `prd-writing`'s evidence, priced): a defensible value story is also the sales team's best artifact.

2. **Get willingness-to-pay evidence beyond asking "would you pay?":** Van Westendorp's four questions (too-cheap / cheap / expensive / too-expensive) for the acceptable RANGE per segment; conjoint analysis when packaging trade-offs matter and volume justifies it; and the strongest evidence — behavior: actual win/loss-by-price-point from sales notes, discount depths that closed vs didn't, upgrade/downgrade flows between existing tiers. Stated intent inflates (`survey-design` step 2's rule — the survey "yes" costs nothing); behavioral evidence is the tier that decides.

3. **Choose the price metric as carefully as the price level — it's the growth contract:** per-seat (simple, budgetable, but taxes collaboration and invites seat-sharing), usage-based (scales with value delivered, but unpredictable bills breed anxiety and procurement friction — pair with caps/commitments), flat/tiered (simplest, but over/under-charges the extremes), hybrid (platform fee + usage — increasingly the B2B default for a reason). The test: does the metric track the VALUE the customer receives, and does it grow when their value grows? A metric misaligned with value generates the resentment tickets ("we pay more for the thing we don't use more of") that churn quietly (`unit-economics-model`'s retention curves feel this first).

4. **Design tiers around segment fences, not feature piles:** 3(±1) tiers mapped to real segments, each fenced by a feature that segment NEEDS and lower segments genuinely don't (SSO/audit-logs/SLA fence enterprise cleanly; gating core value from the entry tier just teaches the market you're not for them); the middle tier as the intended default (anchored by the top tier's existence); upgrade triggers that occur naturally in the product (hitting a limit AT the moment of expansion — the fence the customer climbs willingly). Kano-style classification (`prioritization-frameworks` step 1) sorts which features fence vs which must be everywhere.

5. **Impose discount discipline as structure, not exhortation:** published guardrails (max % by deal size/term, approval tiers above them — `invoice-expense-workflow`'s tiering logic, sales edition); every discount gets something BACK (annual prepay, case study, multi-year, reference — the free discount trains the market that list price is fiction); and the discount log analyzed quarterly by rep/segment (average realized price vs list is the metric — a widening gap is the pricing structure announcing it's wrong somewhere: either list is fantasy or the fences leak).

6. **Test and roll out changes with a safety plan:** grandfather existing customers (or migrate with long notice + a locked-in-rate window — the price change that ambushes the installed base converts a revenue lever into a churn event); new-customer cohorts take the new pricing first (cleanest read: win-rate and realized-price vs the prior cohort — `ab-test-analysis` where volume permits, sequential cohort comparison where it doesn't); pre-commit the evaluation window and the rollback criteria (`experiment-design` step 4's pre-registration — pricing changes have enough politics without post-hoc metric shopping).

7. **Review pricing on a calendar, not a crisis:** annual structural review (value delivered has grown — has price? the product that 3×'d its capability at 2019 prices is donating the delta), quarterly health metrics: realized-vs-list, plan distribution (everyone clustering in one tier = the others are mispriced or misfenced), expansion revenue share, price-driven churn rate, and win/loss-by-price. Pricing is the highest-leverage lever most companies touch least — the calendar is the fix for the touching-least part.

## Common pitfalls
- Cost-plus as strategy: margin-safe and value-blind — underpricing the segment that'd pay 3× and overpricing the one that won't. Cost is the floor's job only (step 1).
- Survey-stated WTP taken literally: the "$50 sounds fair" that converts at $19 — behavioral tiers over stated tiers, always (step 2).
- The everything-tier: entry plan so complete nobody upgrades; the fence audit (step 4) asks what the NEXT segment needs, not what feels generous.
- Metric-value misalignment: per-seat pricing on a product whose value concentrates in two power users per account — seat-sharing isn't customer dishonesty, it's price-structure feedback (step 3).
- Discount anarchy: unlogged, get-nothing-back discounting until realized price is 60% of list and every renewal negotiation starts from the discounted anchor (step 5's structure).
- Set-and-forget: pricing untouched for four years out of fear while the product tripled — the annual review (step 7) makes the conversation routine instead of traumatic; fear of the conversation IS the cost.

## Example
B2B SaaS, symptoms: 85% of customers in the middle tier, realized price 71% of list, sales "everyone demands discounts," pricing untouched three years despite major product expansion. Evidence pass: Van Westendorp by segment (n=240) + 18 months of win/loss-by-price + discount logs. Findings: SMB's acceptable range sat BELOW the entry tier (explaining the discount pressure — list was fantasy for that segment); enterprise's range sat far above the top tier (the 85% middle-clustering was enterprise buying mid-tier happily — money donated); and the price metric (per-seat) fought the product's actual value driver (workflows run, concentrated in few seats — seat counts had flatlined while usage 4×'d). Restructure: hybrid metric (platform fee + workflow-volume bands), entry tier repriced into SMB's measured range (discount guardrails now holdable), enterprise tier rebuilt behind SSO/audit/SLA fences at 2.4× the old top price. Rollout: new cohorts first, 12-month grandfathering with locked renewal for the base; pre-registered evaluation (win-rate, realized price, expansion share, 2 quarters). Results: win-rate flat (the fear was wrong), realized/list 71% → 89%, expansion revenue share doubled on the usage metric, price-driven churn 0.4% — and the enterprise segment's first renewals at the new tier closed without discount. The next review is on the calendar, which is the part that makes it stick.

## Related skills
- `unit-economics-model` — the margin floor and the retention curves pricing moves.
- `competitive-analysis` — the reference frame customers price against.
- `experiment-design` / `ab-test-analysis` — testing changes honestly.
- `survey-design` — WTP instruments that don't lead the witness.
