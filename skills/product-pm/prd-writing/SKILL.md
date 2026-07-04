---
name: prd-writing
description: >
  Use when writing a product requirements document — problem framing,
  explicit non-goals, measurable success criteria, open questions — so
  engineering builds the right thing once. Triggers: "write a PRD",
  "requirements doc for", "spec out this feature", "product brief",
  "review this PRD". NOT for tiny changes (a good ticket suffices) or
  technical design (the engineering design doc this PRD feeds).
---

# PRD Writing

## Overview

A PRD is a set of decisions with evidence attached: a problem statement free of solution vocabulary, non-goals that kill scope creep before it's someone's pet argument, and success criteria that can fail. Every sentence either records a decision or names an open question — length is not the axis, decided-ness is.

## When to Use

- A feature/initiative big enough that multiple people build from a shared document.
- A "simple" request keeps expanding in every conversation — the PRD is the containment vessel.

**When NOT to use:**
- Tiny changes — a well-written ticket.
- Solution architecture — the engineering design doc downstream.

## Prerequisites

- Real evidence of the problem: support tickets, interviews (`user-interview-synthesis`), funnel data, sales-loss notes. A PRD without evidence is fiction with section headings.

## The Workflow

1. **Write the problem statement as user pain + business stake, evidence attached.** Template: *[who] can't [do what] because [obstacle], causing [measured user cost] and [measured business cost].* "Ops managers can't reconcile daily payouts because exports lack fee breakdowns — each does it manually (~4h/week, per 6 interviews), and it's cited in 23% of churn calls." No solution vocabulary here: if the problem statement contains the feature name, it's a justification, not a problem.

2. **Write non-goals immediately after goals — the section that saves the project.** Explicit exclusions with one-line reasons: "Not covering: multi-currency reconciliation (separate problem, <2% of affected users); real-time sync (weekly batch meets the stated need)." Every negotiation the team won't have in week 6 is won here.

3. **Define success as measurable movement, pre-committed:** the 1–2 metrics that should move (`metric-definition` discipline), target and timeframe, guardrails that must NOT degrade, and *how you'll measure* — instrumentation is a requirement, not an afterthought; if the events don't exist, building them is in scope.

4. **Specify requirements as user-outcomes with acceptance criteria, ranked MoSCoW-hard:** each requirement = user story + testable criteria. Must-have list kept brutal — the test: "if this were missing, would we still ship?" Leave the *how* (schema, architecture) to engineering — over-specified PRDs get malicious compliance instead of good solutions.

5. **Keep the open-questions section alive and honest:** every unresolved decision with owner + needed-by date. An empty open-questions section on a v1 PRD signals not completeness but unawareness; the section shrinking across drafts IS the project converging.

6. **Cover the unsexy completeness list** — the items whose absence surfaces in week 9: edge/error states (empty data, permissions, failure modes — `empty-loading-error-states`' design twin), rollout plan (flag? beta? migration for existing users?), i18n/accessibility/compliance constraints, support/docs readiness.

7. **Review by walking the skeptics through, then freeze versions:** engineering reads for feasibility landmines and missing states; design for flow gaps; support/sales for "what will customers actually ask?" Then version the doc — post-freeze changes get a changelog line and a ping to everyone building from it. A silently-edited PRD is how two engineers build two different features.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "Users need a dashboard — that's the problem statement" | That's your solution wearing the problem's chair. The problem is whatever they can't currently see or decide; solution-shaped problems foreclose the better designs (step 1's vocabulary rule). |
| "Non-goals are negative energy — focus on what we're building" | Without them, every stakeholder ships their wishlist via comments, and the Must list triples by design freeze. Non-goals are pre-won arguments; skipping them just reschedules the fights. |
| "Success: improve the experience. We'll know it when we see it" | Unfalsifiable success means the retro can't distinguish a win from a miss, and launch becomes the goal. Pre-committed metrics (step 3) are what make the feature accountable. |
| "Specify the table schema — saves engineering time" | It costs engineering the better design they'd have found, and buys you malicious compliance when the schema fights reality. Outcomes and criteria; the how is theirs. |
| "No open questions — the PRD is complete" | On a v1, an empty section signals unawareness, not completeness. The example's compliance question halved an estimate; unasked, it would have detonated in week 7. |
| "Evidence gathering delays the doc — we know the need" | The example's interviews turned 'invoice customization' into 'logo + tax fields' — a quarter of scope difference. Unevidenced PRDs build the loudest request, not the real one. |

## Red Flags

- The feature's name inside the problem statement.
- No non-goals section, or one added after scope exploded.
- Success criteria with no numbers, no timeframe, or no measurement plan.
- Requirements specifying implementation details.
- The PRD unedited since kickoff while the product diverged.
- Skeptic walkthrough skipped; week-9 questions arriving that support would have asked in week 1.

## Verification

- [ ] Problem statement matches the template with evidence linked — zero solution vocabulary (someone else checks).
- [ ] Non-goals section present with reasons — ≥3 real exclusions.
- [ ] Success metrics pre-committed with targets, guardrails, and instrumentation plan — events named.
- [ ] Must-haves each pass the would-we-still-ship test — list reviewed.
- [ ] Open questions carry owners + dates — none stale past their date.
- [ ] Skeptic walkthrough done (eng/design/support) — attendees and caught-items noted; doc versioned at freeze.

## Example

Request from sales: "we need invoice customization." Evidence pass turned up: 9 lost-deal notes, but interviews (n=8) showed 7 of 9 actually needed *their logo and tax fields* — not templates. PRD problem statement scoped to that; non-goals explicitly listed full template editing ("revisit if logo+tax fields don't move close-rate") and white-labeling. Success: feature cited as resolved blocker in ≥50% of previously-stalled deals within a quarter. Must-haves: 4 items. Open questions caught the compliance one early ("are tax-field formats jurisdiction-dependent? — legal, by design freeze" — answer: yes, EU only, which halved the engineering estimate). Skeptic walkthrough: support asked "what happens to already-sent invoices?" — became an explicit non-goal with comms plan. Shipped in 6 weeks; the deflected template-editor ask returned in Q3 *with its own evidence* and became its own, properly-scoped PRD.

## Related skills

- `user-story-slicing` — decomposing the requirements for delivery.
- `metric-definition` — the success-criteria rigor in step 3.
- `feature-scoping-cut` — when the Must list still won't fit the timeline.
- `user-interview-synthesis` — producing the evidence step 1 requires.
