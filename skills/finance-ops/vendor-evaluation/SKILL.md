---
name: vendor-evaluation
description: >
  Use when selecting or renewing a vendor — total cost of ownership beyond
  the sticker, contract terms that matter, lock-in pricing, negotiation
  levers. Triggers: "evaluate this vendor", "compare these vendors",
  "renewal negotiation coming up", "TCO analysis", "contract review for
  this SaaS", "are we overpaying".
---

# Vendor Evaluation

## When to use this skill
- Selecting a vendor for meaningful spend, or preparing a renewal negotiation.
- The renewals calendar (`invoice-expense-workflow` step 3) surfaced a contract worth re-examining.
- NOT for evaluating open-source/technology adoption on technical merit — that's `technology-evaluation`; this skill owns the commercial layer (the two run together for paid developer tools).

## Prerequisites
- The requirement stated vendor-independently, with must-haves vs wants (same discipline as `technology-evaluation` step 1 — requirements written after the demo inherit the demo).
- Current true spend if it's a renewal (all SKUs, overages, adjacent costs — pulled from AP, not from memory).

## Workflow

1. **Compute TCO over the full term, not the sticker:** license/subscription + implementation and migration cost (one-time, real, routinely 0.5–2× year-one fees) + integration build/maintenance + training and productivity dip + internal admin time + the exit cost you'll eventually pay (data extraction, migration off — priced NOW, `technology-evaluation` step 4's same move). Compare vendors on 3-year TCO; the cheaper-sticker vendor with the expensive implementation and proprietary data format loses on the honest math surprisingly often.

2. **Structure the comparison against your criteria, weighted before demos:** must-haves as pass/fail gates (compliance certs, data residency, SSO — non-negotiables screened first, cheaply), weighted wants scored after hands-on evaluation (trial/POC with YOUR data and YOUR workflows, not the demo's golden path — vendor demos are `technology-evaluation` step 5's tutorial-path problem with a salesperson attached). Reference checks with actual customers at your scale — asked specifically "what surprised you post-purchase?" and "what would you negotiate differently?" (the two questions that produce information instead of testimonials).

3. **Read the contract for the terms that bite, not just the price:**
   - **Renewal mechanics:** auto-renew windows (the 90-day notice period that silently re-commits you — calendar it on day one), renewal price caps (uncapped renewals = year-one discount, year-three hostage; cap increases at CPI or a fixed % IN the contract).
   - **Exit and data:** termination rights, data export format and assistance obligations, deletion certification.
   - **Usage definitions:** what counts as a "user"/"transaction"/"record" (the ambiguous unit is the vendor's future upsell), overage rates, true-up mechanics.
   - **Liability and SLA:** actual remedies (service credits nobody claims vs real termination rights on chronic breach), support tiers' actual response commitments.
   - **The one-way ratchets:** clauses letting them reduce service/change terms unilaterally — flagged and negotiated.

4. **Build negotiation leverage deliberately — it's mostly timing and alternatives:** a live competitive alternative (even a partially-evaluated one — the BATNA that's real, not bluffed), their quarter/year-end (vendor sales calendars are your discount calendar), multi-year and prepay traded FOR something (never given free — `pricing-analysis` step 5 from the buyer's side), volume honesty (buy the seats you'll use in 6 months, not the optimistic 3-year count — shelfware is the vendor's margin and your `budget-variance-analysis` finding), and the reference/case-study card (worth real percentage points to vendors, costs you little).

5. **Negotiate the terms that outlive the discount:** the renewal cap is worth more than an extra 5% off year one (the discount expires; the cap protects every future year); notice-period alignment (renewal notice window ≥ your evaluation time); price protection on growth (locked unit rates for expansion — the growth-punishing contract taxes your success); and downgrade rights (the right to reduce seats/tier at renewal — the asymmetry vendors never offer unprompted). Get every verbal promise into the order form; the sales engineer's "of course that's included" has a half-life of one org chart change.

6. **Decide with the score visible, document as a light ADR:** weighted scores + TCO + the contract-terms delta, decision recorded with drivers ranked and losers treated fairly (`architecture-decision-record` — the "why not X?" question arrives at every renewal), and the leash where risk warrants: pilot scope with success criteria and expansion contingent on them (`technology-evaluation` step 7's adopt-on-a-leash, commercial edition).

7. **Manage the relationship as an asset with a calendar:** renewal prep starts 120+ days out (leverage evaporates inside the notice window — the calendared date from step 3 is when ALTERNATIVES get refreshed, not when panic starts), usage-vs-entitlement reviewed quarterly (reclaim shelfware before true-up, catch overage trends before they're invoices), SLA/credit claims actually filed (unclaimed credits train the vendor that the SLA is decorative), and the relationship-capital ledger honest: the vendor who flexed for you in the tight week (`cash-flow-forecast` step 3's levers) earned renewal goodwill — and vice versa.

## Common pitfalls
- Sticker-price comparison: vendor A "cheaper" until implementation, integration maintenance, and exit costs load in — the 3-year TCO reversal is the norm, not the exception (step 1).
- Demo-based selection: the golden-path demo scoring 9/10 while your actual workflow hits the product's weak wing in week two — POC with your data or the evaluation is theater (step 2).
- The unread auto-renew: 90-day notice discovered on day 60 — a year re-committed by calendar negligence. Day-one calendaring (step 3) is a two-minute control.
- Uncapped renewals accepted for a year-one discount: the 40% renewal increase three years in, with your data deeply embedded — the cap (step 5) was worth more than the discount, every time.
- Optimistic volume commits: 500 seats bought on the growth plan, 210 used at true-up — shelfware is pure vendor margin; buy the 6-month number with locked expansion rates (step 4).
- Renewal prep inside the notice window: no alternative refreshed, no leverage, list-price increase accepted with a sigh — the 120-day calendar (step 7) is the entire negotiation position.

## Example
Renewal inbound: analytics platform, $180k/yr, vendor opening with a 22% increase, 90-day notice window. Prep started at day 130 (calendar control from the prior cycle). Usage review: 340 of 500 licensed seats active — the shelfware became the opening counter. Alternative refreshed: a two-week POC with the runner-up from the original evaluation (real enough to reference, `technology-evaluation` shared the work). Contract read flagged the uncapped-renewal clause as THE target — the 22% ask was the clause working as the vendor designed. Negotiation, timed to their fiscal Q4: seats right-sized to 375 with locked expansion pricing, renewal increases capped at 5%/yr for the new 2-year term, downgrade rights at term end, one executive case study traded in — landing at a 4% net increase on right-sized volume (vs the 22% on inflated volume: ~$61k/yr delta). The unclaimed-SLA audit found $9k of never-filed credits — filed, recovered, and the filing itself repriced the vendor's attention to their own SLA. Decision memo'd as a light ADR; the 120-day calendar entry for the NEXT cycle created before the ink dried.

## Related skills
- `technology-evaluation` — the technical-merit sibling; run together for tooling.
- `invoice-expense-workflow` — the renewals calendar and approval path.
- `pricing-analysis` — the vendor's playbook, read from the other side.
- `cash-flow-forecast` — payment-timing levers and prepay trade-offs.
