---
name: prd-writing
description: >
  Use when writing a product requirements document — problem framing,
  explicit non-goals, measurable success criteria, open questions — so
  engineering builds the right thing once. Triggers: "write a PRD",
  "requirements doc for", "spec out this feature", "product brief",
  "review this PRD".
---

# PRD Writing

## When to use this skill
- A feature/initiative is big enough that multiple people will build from a shared document.
- A "simple" request keeps expanding in every conversation — the PRD is the containment vessel.
- NOT for tiny changes (a well-written ticket suffices) or technical design (that's the engineering design doc the PRD feeds — solution architecture doesn't belong here).

## Prerequisites
- Real evidence of the problem: support tickets, user interviews (`user-interview-synthesis`), funnel data, sales-loss notes. A PRD without evidence is a fiction with section headings.

## Workflow

1. **Write the problem statement as user pain + business stake, evidence attached.** Template: *[who] can't [do what] because [obstacle], causing [measured user cost] and [measured business cost].* "Ops managers can't reconcile daily payouts because exports lack fee breakdowns — each does it manually (~4h/week, per 6 interviews), and it's cited in 23% of churn calls." No solution vocabulary in this section: if the problem statement contains the feature name, it's a justification, not a problem.

2. **Write non-goals immediately after goals — the section that saves the project.** Explicit exclusions with one-line reasons: "Not covering: multi-currency reconciliation (separate problem, <2% of affected users); real-time sync (weekly batch meets the stated need)." Non-goals are where scope creep goes to die *before* it's someone's pet argument in week 6; every negotiation the team won't have later is won here.

3. **Define success as measurable movement, pre-committed:** the 1–2 metrics that should move (`metric-definition` discipline — precise numerator/denominator), the target and timeframe, the guardrails that must NOT degrade, and *how you'll measure* (instrumentation is a requirement, not an afterthought — if the events don't exist, building them is in scope). "Success: manual reconciliation time self-reported <30min/week for 70% of active ops users within 60 days of GA; guardrail: export latency p95 unchanged."

4. **Specify requirements as user-outcomes with acceptance criteria, ranked MoSCoW-hard:** each requirement = user story + testable criteria ("export includes per-transaction fee breakdown; ops user can reconcile a sample day without opening another tool"). Must-have list kept brutally short — the test: "if this were missing, would we still ship?" Everything surviving that test twice is a Should. Leave the *how* (schema, architecture) to engineering — over-specified PRDs get malicious compliance instead of good solutions.

5. **Keep the open-questions section alive and honest:** every unresolved decision listed with owner + needed-by date ("Do agencies need per-client exports? — @maya, before design freeze"). An empty open-questions section on a v1 PRD signals not completeness but unawareness; the section shrinking over drafts IS the project converging.

6. **Cover the unsexy completeness list** — the items whose absence surfaces in week 9: edge/error states (empty data, permissions, failure modes — `empty-loading-error-states` is the design-side twin), rollout plan (flag? beta cohort? migration for existing users? — `deployment-rollback-plan` thinking at product level), i18n/accessibility/compliance constraints, and support/docs readiness (`release-notes` owes its inputs to this section).

7. **Review by walking the skeptics through, then freeze versions:** engineering reads for feasibility land-mines and missing states; design for flow gaps; support/sales for "what will customers actually ask?" Incorporate, then version the doc — post-freeze changes get a changelog line and a ping to everyone building from it (a silently-edited PRD is how two engineers build two different features).

## Common pitfalls
- Solution masquerading as problem ("Users need a dashboard" — that's your solution; the problem is whatever they can't currently see/decide). The step-1 no-solution-vocabulary rule exists for this.
- No non-goals section — every stakeholder ships their wishlist into the requirements via comments, and the Must list triples by design freeze.
- Success = "improve the experience" / launch = success. Unfalsifiable success criteria mean the retro can't distinguish a win from a miss (`ab-test-analysis` pre-registration logic applies to features too).
- Requirements specifying implementation ("store fees in a new fees table") — now engineering is choosing between a bad design and contradicting the PRD.
- The 30-page PRD nobody reads vs. the 1-pager hiding all the hard decisions in ambiguity — length isn't the axis; *decided-ness* is. Every sentence either records a decision or names an open question.
- Written once, never updated: the PRD says X, the product does Y, and the next PM inherits archaeology. Living doc until GA, then explicitly archived.

## Example
Request from sales: "we need invoice customization." Evidence pass turned up: 9 lost-deal notes, but interviews (n=8) showed 7 of 9 actually needed *their logo and tax fields* — not templates. PRD problem statement scoped to that; non-goals explicitly listed full template editing ("revisit if logo+tax fields don't move close-rate") and white-labeling. Success: feature cited as resolved blocker in ≥50% of previously-stalled deals within a quarter; guardrail: invoice render time. Must-haves: 4 items. Open questions section caught the compliance one early ("are tax-field formats jurisdiction-dependent? — legal, by design freeze" — answer: yes, EU only, which halved the engineering estimate by descoping US formats to Should). Skeptic walkthrough: support asked "what happens to already-sent invoices?" — became an explicit non-goal with comms plan. Shipped in 6 weeks; the deflected template-editor ask returned in Q3 *with its own evidence* and became its own, properly-scoped PRD. Two projects, both small, instead of one unbounded one.

## Related skills
- `user-story-slicing` — decomposing the requirements for delivery.
- `metric-definition` — the success-criteria rigor in step 3.
- `feature-scoping-cut` — when the Must list still won't fit the timeline.
- `user-interview-synthesis` — producing the evidence step 1 requires.
