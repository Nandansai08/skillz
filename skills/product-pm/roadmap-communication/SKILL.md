---
name: roadmap-communication
description: >
  Use when communicating a roadmap outward — now/next/later framing,
  committing to problems instead of dates, handling "when will X ship"
  pressure without lying. Triggers: "share the roadmap", "customer asked
  for a roadmap", "roadmap slide", "when should we promise this", "sales
  wants dates". NOT for deciding what's ON the roadmap (see
  prioritization-frameworks) — this communicates the output without
  converting estimates into contracts.
---

# Roadmap Communication

## Overview

A roadmap is honest exactly when its certainty language narrows as the horizon extends — Now concrete, Next as committed problems, Later as direction. Every date given without a real constraint behind it is a future apology on a payment plan.

## When to Use

- Presenting the roadmap to customers, sales, execs, or the company.
- The current roadmap doc has become a promise-tracking liability.

**When NOT to use:**
- Deciding contents — `prioritization-frameworks`.

## Prerequisites

- A prioritized plan that actually exists.
- Knowledge of the audiences and what each does with roadmap information (sales sells it, customers plan around it, engineers commit to it).

## The Workflow

1. **Structure as Now / Next / Later — with the semantics stated on the slide:**
   - **Now:** in active development; high confidence; described concretely.
   - **Next:** committed *problems*, sequenced; shapes may change; no dates.
   - **Later:** directional intent; explicitly subject to reordering; problem areas only.
   The gradient IS the honesty mechanism: a roadmap describing December's work as concretely as this sprint's is lying about one of them.

2. **Frame items as problems-being-solved, not feature nouns:** "Cut payout-reconciliation time for ops teams" survives solution changes, communicates the *why* that sales and customers actually retell, and keeps design latitude open (`prd-writing` step 1's discipline, projected outward).

3. **Commit dates only where dates are real constraints, and label them:** compliance deadlines, contracted deliverables, event launches get dates (they're constraints). Everything else gets horizon language. When a date must be given under pressure: give it with its confidence and its dependency ("Q4, high confidence, assuming the auth migration lands — that's the risk"). One honest caveat outperforms the silent slip it prevents.

4. **Version the artifact per audience, from one source of truth:** internal sees everything including risky bets; sales gets Now/Next with talk-tracks and explicit *do-not-promise* markings — sales WILL forward whatever they receive; write every word as if the customer reads it, because they will; public shows themes and Now-column concreteness only. Divergent hand-maintained decks drift into contradiction.

5. **Handle "when will X ship?" with the three-part move:** honest status ("Next — sequenced after reconciliation, because [reason]"), the underlying-need question ("what's driving the timing for you?" — often the real need is smaller/sooner: a workaround, an API field), and a concrete follow-up that isn't a ship date ("I'll flag you when it enters design — you're in the beta"). What never works: the soft "probably next quarter" that the asker records as a promise.

6. **Publish changes as first-class communication:** when Later reorders or Next slips, announce the delta with the reason to everyone who saw the previous version. Silent roadmap edits are how trust dies retroactively — the customer who planned around the old slide finds out at renewal. Predictable cadence: monthly internal, quarterly external.

7. **Track the promises the roadmap creates, deliberately:** every dated commitment and customer-specific "you'll have it by" in one visible list with owners — the roadmap's *liabilities*. If the liability list grows faster than the shipped list, the communication practice (usually step 3 discipline) is the bug.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "Sales needs dates to close — just put quarters on everything" | The Gantt-roadmap's every cell is a future apology, and the apologies compound into distrust of ALL roadmap communication. The gradient + labeled real constraints is what sales can actually defend at renewal. |
| "Feature names are clearer than problem statements" | 'AI Assistant, Q3' locks a solution before design and makes shipping something better 'missing the roadmap.' Problems survive solution changes; nouns become handcuffs. |
| "One deck for everyone — single source of truth" | Single SOURCE, filtered views (step 4). The unfiltered deck either hands sales the risky internals (which get sold) or hands engineering the sanitized fiction (which gets planned against). |
| "'Probably next quarter' keeps the customer happy for now" | The asker records it as a commitment, and 'probably' doesn't survive their procurement notes. The three-part move gives them something real without minting a liability. |
| "The reorder is internal — no need to announce it" | The customer who planned around the old slide discovers the change at renewal, which converts a reasonable reprioritization into a betrayal narrative. Deltas are communications, not edits. |
| "We don't do dates, period — it's the honest policy" | Stakeholders need planning signals; pure refusal reads as evasion and pushes them to invent their own dates. Labeled confidence + real-constraint dates is honest AND usable. |

## Red Flags

- Quarter-precision dates on items 10 months out.
- The roadmap reshaped after each big-customer call.
- Sales decks containing items marked (or that should be marked) do-not-promise.
- Stated-confidence items slipping repeatedly with labels unchanged.
- No record of what was promised to whom.
- Customers citing roadmap slides the team no longer recognizes.

## Verification

- [ ] Now/Next/Later semantics printed on the artifact itself.
- [ ] Items phrased as problems; feature nouns only in Now — reviewed.
- [ ] Every date on the roadmap traceable to a real constraint — list checked.
- [ ] Per-audience versions generated from one source; sales version carries do-not-promise markings — links.
- [ ] Change-delta communications sent for the last reorder — link.
- [ ] Liability list current with owners — reviewed this cycle.

## Example

Sales escalation: enterprise prospect "needs the roadmap with dates to sign." Old artifact was a feature Gantt already two apologies deep. Rebuild: Now (3 items, concrete), Next (4 problems with sequencing reasons), Later (3 themes); semantics footer on every page. The prospect call used step 5: their "need SSO by Q2" unpacked into "security review requires SAML *documentation* by Q2" — satisfied with the existing SAML beta docs + a beta seat, no date promised; deal signed. Sales version shipped with do-not-promise markings and a talk-track per item; one AE forwarded it to a customer within a week (as predicted), consequence-free because it was written for that. Quarterly change post moved one Next item down with the churn-data reason — the customer who'd asked about it referenced the *reason* approvingly in their renewal call. Liability list after two quarters: 3 dated commitments (all compliance-real), zero soft promises — down from the old deck's 14.

## Related skills

- `prioritization-frameworks` — the decisions this artifact communicates.
- `stakeholder-update` — the recurring-communication sibling.
- `prd-writing` — problem-framing at document scale.
- `release-notes` — announcing the Now column as it ships.
