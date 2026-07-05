---
name: vendor-evaluation
description: >
  Use when selecting or renewing a vendor — total cost of ownership beyond
  the sticker, contract terms that matter, lock-in pricing, negotiation
  levers. Triggers: "evaluate this vendor", "compare these vendors",
  "renewal negotiation coming up", "TCO analysis", "contract review for
  this SaaS", "are we overpaying". NOT for open-source adoption on
  technical merit (see technology-evaluation) — this skill owns the
  commercial layer.
---

# Vendor Evaluation

## Overview

The sticker price is the vendor's opening argument; 3-year TCO, the renewal-cap clause, and your calendared notice window are yours. Leverage is mostly timing and alternatives — and it evaporates inside the notice window, which is why renewal prep starts at day 120, not day 60.

## When to Use

- Selecting a vendor for meaningful spend, or preparing a renewal negotiation.
- The renewals calendar surfaced a contract worth re-examining.

**When NOT to use:**
- Technical-merit adoption of tools — `technology-evaluation` (the two run together for paid dev tools).

## Prerequisites

- The requirement stated vendor-independently, must-haves vs wants.
- Current true spend if a renewal (all SKUs, overages — pulled from AP, not memory).

## The Workflow

1. **Compute TCO over the full term, not the sticker:** subscription + implementation/migration (routinely 0.5–2× year-one fees) + integration build/maintenance + training and productivity dip + admin time + the exit cost you'll eventually pay (priced NOW — `technology-evaluation` step 4's same move). Compare on 3-year TCO; the cheaper-sticker vendor with the proprietary format loses on honest math surprisingly often.

2. **Structure the comparison against your criteria, weighted before demos:** must-haves as pass/fail gates; weighted wants scored after hands-on evaluation (POC with YOUR data and YOUR workflows — vendor demos are the tutorial-path problem with a salesperson attached). Reference checks with customers at your scale, asking the two questions that produce information: "what surprised you post-purchase?" and "what would you negotiate differently?"

3. **Read the contract for the terms that bite, not just the price:**
   - **Renewal mechanics:** auto-renew notice windows (calendar them DAY ONE — the 90-day window discovered on day 60 re-commits a year by negligence); renewal price caps (uncapped = year-one discount, year-three hostage; cap at CPI or fixed % IN the contract).
   - **Exit and data:** termination rights, export format and assistance, deletion certification.
   - **Usage definitions:** what counts as a "user"/"transaction" (the ambiguous unit is the vendor's future upsell), overage rates, true-up mechanics.
   - **SLA remedies:** real termination rights on chronic breach vs credits nobody claims.
   - **One-way ratchets:** unilateral service/term changes — flagged and negotiated.

4. **Build negotiation leverage deliberately — it's mostly timing and alternatives:** a live competitive alternative (real, not bluffed); their fiscal quarter/year-end (vendor sales calendars are your discount calendar); multi-year and prepay traded FOR something, never given free; volume honesty (buy the 6-month number, not the optimistic 3-year count — shelfware is the vendor's margin and your variance finding); the reference/case-study card (worth real points, costs little).

5. **Negotiate the terms that outlive the discount:** the renewal cap is worth more than an extra 5% off year one (the discount expires; the cap protects every future year); notice-period ≥ your evaluation time; locked expansion pricing (the growth-punishing contract taxes your success); downgrade rights at renewal. Get every verbal promise into the order form — "of course that's included" has a half-life of one org chart change.

6. **Decide with the score visible, document as a light ADR:** weighted scores + TCO + contract-terms delta, drivers ranked, losers treated fairly (`architecture-decision-record` — "why not X?" arrives at every renewal); a leashed pilot where risk warrants.

7. **Manage the relationship as an asset with a calendar:** renewal prep at 120+ days (leverage evaporates inside the notice window — the calendared date is when ALTERNATIVES get refreshed, not when panic starts); usage-vs-entitlement quarterly (reclaim shelfware before true-up); SLA credits actually filed (unclaimed credits train the vendor that the SLA is decorative); the relationship-capital ledger honest both directions.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "Vendor A is $40k cheaper — clear winner" | On sticker. Add implementation, integration maintenance, and the proprietary-format exit, and the 3-year TCO reversal is the norm. Honest math or the discount buys the moat. |
| "The demo covered everything we need" | The demo is the golden path with a salesperson steering. The POC with your data is where the weak wing shows — before the contract, not in week two. |
| "Take the bigger year-one discount over the renewal cap" | The discount expires in twelve months; the uncapped renewal prices every year after, with your data embedded as the hostage. The cap outearns the discount in every multi-year case. |
| "Buy the 500 seats — we'll grow into them" | 210 active at true-up: shelfware is pure vendor margin. Buy the 6-month number with locked expansion pricing; growth optimism is the sales rep's asset, not yours. |
| "Renewal's not until Q3 — plenty of time" | Prep inside the notice window = no refreshed alternative = no leverage = list-price increase accepted with a sigh. Day-120 is the negotiation; day-60 is the surrender. |
| "Filing SLA credits nickel-and-dimes the relationship" | Unclaimed credits teach the vendor their SLA is decorative — and the example's $9k filing repriced their attention to it. The ledger works both directions; so does respect. |

## Red Flags

- No TCO computation; sticker comparison only.
- Auto-renew dates living in nobody's calendar.
- Uncapped renewal language in a signed contract.
- Entitlements vs usage never reviewed; true-ups arriving as surprises.
- Verbal promises absent from the order form.
- Renewal negotiations starting inside the notice window.

## Verification

- [ ] 3-year TCO per finalist including exit cost — table attached.
- [ ] Criteria weighted before demos; POC run on your data — findings noted.
- [ ] Contract-terms checklist completed (renewal mechanics, exit, units, SLA, ratchets) — per-term dispositions.
- [ ] Notice windows calendared at signature — entries linked.
- [ ] Negotiated terms include caps/expansion-locks/downgrade rights where applicable; verbals in the order form — contract cited.
- [ ] Decision ADR'd with ranked drivers.
- [ ] Renewal-prep trigger at ≥120 days with the alternatives-refresh task attached.

## Example

Renewal inbound: analytics platform, $180k/yr, vendor opening with a 22% increase, 90-day notice window. Prep started at day 130 (calendar control from the prior cycle). Usage review: 340 of 500 licensed seats active — shelfware became the opening counter. Alternative refreshed: a two-week POC with the original runner-up (real enough to reference). Contract read flagged the uncapped-renewal clause as THE target — the 22% ask was the clause working as designed. Negotiation, timed to their fiscal Q4: seats right-sized to 375 with locked expansion pricing, renewals capped at 5%/yr for the new term, downgrade rights at term end, one case study traded in — landing at 4% net on right-sized volume (vs 22% on inflated volume: ~$61k/yr delta). The unclaimed-SLA audit found $9k of never-filed credits — filed, recovered, and the filing repriced the vendor's attention to their own SLA. Decision memo'd; the day-120 calendar entry for the NEXT cycle created before the ink dried.

## Related skills

- `technology-evaluation` — the technical-merit sibling; run together for tooling.
- `invoice-expense-workflow` — the renewals calendar and approval path.
- `pricing-analysis` — the vendor's playbook, read from the other side.
- `cash-flow-forecast` — payment-timing levers and prepay trade-offs.
