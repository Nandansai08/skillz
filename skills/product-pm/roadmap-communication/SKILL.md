---
name: roadmap-communication
description: >
  Use when communicating a roadmap outward — now/next/later framing,
  committing to problems instead of dates, handling the "when will X ship"
  pressure without lying. Triggers: "share the roadmap", "customer asked
  for a roadmap", "roadmap slide", "when should we promise this",
  "sales wants dates".
---

# Roadmap Communication

## When to use this skill
- Presenting the roadmap to customers, sales, execs, or the company.
- The current roadmap doc has become a promise-tracking liability.
- NOT for deciding what's ON the roadmap — that's `prioritization-frameworks`; this skill communicates the output without converting estimates into contracts.

## Prerequisites
- A prioritized plan that actually exists (this skill can't communicate decisions that haven't been made).
- Knowledge of the audiences and what each does with roadmap information (sales sells it, customers plan around it, engineers commit to it — same artifact, three different blast radii).

## Workflow

1. **Structure as Now / Next / Later — with the semantics stated on the slide:**
   - **Now:** in active development; high confidence; described concretely.
   - **Next:** committed *problems*, sequenced; shapes may change; no dates.
   - **Later:** directional intent; explicitly subject to reordering; described as problem areas only.
   The gradient IS the honesty mechanism: certainty language narrows as the horizon extends. A roadmap that describes December's work as concretely as this sprint's is lying about one of them.

2. **Frame items as problems-being-solved, not feature nouns:** "Cut payout-reconciliation time for ops teams" travels better than "Fee-breakdown export v2" — it survives solution changes (you can swap the implementation without "missing a roadmap item"), it communicates the *why* that sales and customers actually retell, and it keeps design latitude open (`prd-writing` step 1's discipline, projected outward).

3. **Commit dates only where dates are real constraints, and label them:** compliance deadlines, contracted deliverables, event-driven launches get dates (they're constraints — `prioritization-frameworks` step 5). Everything else gets horizon language. When a date must be given under pressure, give it with its confidence and its dependency ("Q4, high confidence, assuming the auth migration lands — that's the risk"). One honest caveat outperforms the silent slip it prevents.

4. **Version the artifact per audience, from one source of truth:** internal/engineering sees everything including the risky bets; sales gets Now/Next with talk-tracks and explicit *do-not-promise* markings (sales WILL forward whatever they receive — write every word as if the customer reads it, because they will); public/customer version shows themes and Now-column concreteness only. Divergent hand-maintained decks drift into contradiction — one source, filtered views.

5. **Handle "when will X ship?" with the three-part move:** the honest status ("Next — sequenced after reconciliation, because [reason from the prioritization]"), the underlying-need question ("what's driving the timing for you?" — often the real need is smaller/sooner than the feature: a workaround, an API field, a report), and a concrete follow-up commitment that isn't a ship date ("I'll flag you when it enters design — you'll be in the beta"). What never works long-term: the soft "probably next quarter" that the asker records as a promise.

6. **Publish changes as first-class communication:** when Later reorders or Next slips, announce the delta with the reason ("moved down because churn data pointed at X — here's what moved up") to everyone who saw the previous version. Silent roadmap edits are how trust dies retroactively — the customer who planned around the old slide finds out at renewal. Change-log discipline (`changelog-writing` energy, roadmap edition), on a predictable cadence (monthly internal, quarterly external).

7. **Track the promises the roadmap creates, deliberately:** every dated commitment and every customer-specific "you'll have it by" lives in one visible list with owners — because these are the roadmap's *liabilities*, and unmanaged they surface as surprise escalations. If the liability list grows faster than the shipped list, the communication practice (usually step 3 discipline) is the bug.

## Common pitfalls
- The Gantt-roadmap: 14 features with quarter-precision dates, 10 months out. Every cell is a future apology; the format itself promises what estimation can't deliver.
- Feature-noun roadmaps that lock solutions before design ("AI Assistant, Q3") — now shipping something better is "missing the roadmap."
- One deck for all audiences: either sales gets the risky internals (and sells them), or engineering gets the sanitized version (and plans against fiction).
- Answering date pressure with silence or "we don't do dates" purity — stakeholders NEED planning signals; the gradient + labeled-confidence approach (steps 1, 3) is the alternative to both lying and stonewalling.
- Roadmap-by-loudest-renewal: the artifact quietly reshaped after each big-customer call, until it's a promise ledger, not a strategy (`prioritization-frameworks` step 7's triage lane is the pressure valve).
- Never revisiting stated confidence: "high confidence" items slipping twice per year means the confidence labels are decorative — calibrate them against the track record, publicly.

## Example
Sales escalation: enterprise prospect "needs the roadmap with dates to sign." Old artifact was a feature Gantt already two apologies deep. Rebuild: Now (3 items, concrete, incl. the reconciliation work), Next (4 problems with sequencing reasons), Later (3 themes); semantics footer on every page. The prospect call used step 5: their "need SSO by Q2" unpacked into "security review requires SAML *documentation* by Q2" — satisfied with the existing SAML beta docs + a beta seat, no date promised; deal signed. Sales version shipped with do-not-promise markings and a talk-track per item; one AE forwarded it to a customer within a week (as predicted), consequence-free because it was written for that. Quarterly change post moved one Next item down with the churn-data reason — the customer who'd asked about it referenced the *reason* approvingly in their renewal call. Liability list after two quarters: 3 dated commitments (all compliance-real), zero soft promises — down from the old deck's 14.

## Related skills
- `prioritization-frameworks` — the decisions this artifact communicates.
- `stakeholder-update` — the recurring-communication sibling.
- `prd-writing` — problem-framing at document scale.
- `release-notes` — announcing the Now column as it ships.
