---
name: financial-model-review
description: >
  Use when auditing a spreadsheet financial model — hunting hardcodes,
  broken links, circular logic, and heroic assumptions; testing whether
  outputs survive sensitivity. Triggers: "review this model", "check the
  spreadsheet before the board sees it", "does this model hold up",
  "stress test these projections", "the numbers feel too good".
---

# Financial Model Review

## When to use this skill
- A model (budget, forecast, deal case, fundraise deck's engine) is about to drive a real decision and needs an audit.
- Two models of the same business disagree and someone must arbitrate.
- NOT for building the model — this is the adversarial pass; and the spreadsheet mechanics generalize (the same review shape applies to a Python model, where `coverage-analysis`-style discipline adds tests).

## Prerequisites
- The live model file (not a PDF of it — you audit formulas, not printouts), its stated purpose, and which outputs actually feed the decision.

## Workflow

1. **Map the architecture before touching any number:** inputs/assumptions → calculations → outputs — are they SEPARATED (tabs or clearly-zoned areas), or is the model a lasagna of all three? Trace the 2–3 decision-driving outputs backward to their assumption roots (trace precedents / F5-Special tooling): the paths you walk are the audit's spine; a model whose outputs can't be traced in an hour is failing the review at the architecture layer regardless of its arithmetic.

2. **Hunt hardcodes — the spreadsheet's most reliable pathology:** numbers typed inside formulas (`=B4*1.2*0.85` — what is 0.85, who approved it, and why doesn't it live in the assumptions tab where it can be seen and changed?); the mechanical sweep: formula-auditing tools or a pass flagging constants inside formulas, plus the visual scan for the classic tell — a row of formulas with one hand-overridden cell mid-range (the "plug" someone typed to make Q3 tie out, which now silently disconnects Q3 from every assumption). Every hardcode found is either promoted to a labeled assumption or explained in a comment; unexplained plugs are findings.

3. **Check the model's structural integrity:**
   - **Circularity:** intentional (interest-on-average-debt) — documented with iteration settings noted — vs accidental (a reference loop someone silenced by toggling iterative calc). Accidental circularity is a correctness bug wearing a settings band-aid.
   - **Cross-sheet and external links:** broken references, links to files on someone's desktop, ranges that didn't extend when rows were added (the SUM that stops at row 40 of a 60-row list — the off-by-a-range error is spreadsheet-land's silent classic).
   - **Consistency of time:** every column the same period convention, no month/quarter splices; flags for period boundaries (the model mixing calendar and fiscal years mid-row).
   - **Balance checks:** does the balance sheet balance BY FORMULA with a visible check cell, do cash flows tie to cash balances — a model without built-in check cells expects its reader to supply the paranoia it lacks.

4. **Audit the assumptions against evidence — where models actually lie:** each key input gets the interrogation: source? (actuals-derived, market data, or wish — `source-credibility-check` for the external ones); consistent with history? (the model growing 8%/month when the last 12 actuals averaged 3% owes an explanation named in the model, not in the meeting); internally consistent? (revenue tripling while headcount is flat and CAC constant — the assumptions contradict each other; growth costs something, `unit-economics-model`'s CAC-saturation reality). The hockey stick isn't automatically wrong — the undischarged burden of explaining it is.

5. **Run the sensitivity pass — the review's decision-grade output:** flex the 3–5 assumptions the step-1 trace showed carry the outputs (±20%, and to their plausible-worst from history): which single assumption flips the decision? (that one gets the scrutiny and the monitoring); does the model break mechanically under stress (negative cash the formulas ignore, growth assumptions that quietly imply >100% market share by year 4)? A model whose verdict survives sensitivity is information; one whose verdict dies at −10% on one input is a bet on that input — and the review's job is saying so in exactly those words.

6. **Reconcile against reality anchors:** back-test — feed the model last year's inputs and compare its outputs to what actually happened (the model that can't retrodict shouldn't be trusted to predict); tie current-period model figures to the accounting actuals (`budget-variance-analysis`'s basis-check cousin); and sanity-scale the outputs (implied market share, revenue per employee, margins vs any comparable — the outside view that catches what cell-level auditing can't).

7. **Report findings by severity with the fix-path, and leave the model better:** blockers (errors changing the decision: the broken range, the contradictory assumptions), material (hardcodes on load-bearing paths, missing sensitivity), hygiene (structure, labeling); plus the two artifacts the model should keep: a check-cell dashboard (all balance/tie checks visible in one place) and a change-log-plus-version discipline going forward ("board_v3_FINAL_v2 (1).xlsx" is a filename and a warning). The review that only produces a memo fixed nothing; the one that leaves check cells behind keeps auditing after you've left.

## Common pitfalls
- Auditing outputs for plausibility while skipping formula mechanics: the deck's numbers "look reasonable" — and the SUM stopping at row 40 is why. Plausible and correct are independent properties; check both.
- The reviewer captured by the model's frame: interrogating cell logic while accepting the architecture's buried premise (the TAM tab everyone scrolls past). Step 4's outside-view questions attack premises, not just formulas.
- Hardcode whack-a-mole without asking why: plugs cluster where the model fights its author — three plugs in the same section mean the section's logic is wrong, not that three cells need promoting.
- Sensitivity theater: the tornado chart flexing trivial inputs while the load-bearing assumption (retention, win-rate) sits untouched because flexing it is "unrealistic" — the assumption too scary to flex is the review's headline (step 5).
- Politeness pricing: material findings softened for the model's owner until the memo reads as hygiene notes — the board then decides on the unflagged blocker. Severity-honest or the review is complicity (`code-review-checklist`'s severity discipline, money edition).
- One-time review of a living artifact: the audited model mutates the following week, unversioned — the check-cell dashboard and change log (step 7) are what survive.

## Example
Fundraise model headed to a term-sheet negotiation; founder asks for a hostile read. Architecture map: assumptions reasonably separated, BUT the revenue engine traced back to a "Pipeline" tab last reconciled two quarters ago. Hardcode sweep: 23 constants-in-formulas; 19 benign, 3 material (a typed 0.92 "gross margin uplift" nobody could source; two Q3 plugs where model met reality and reality lost), 1 blocker — a growth formula overridden for exactly the four quarters shown in the deck's chart, reconnecting to assumptions only after the chart's window ended. Structural pass: one broken range (new products added below a SUM's reach — revenue understated $400k, ironically favorable) and no check cells anywhere. Assumption audit: month-over-month growth at 2.7× trailing actuals, justified nowhere; sensitivity showed the raise's runway math flipped at −15% on that single input. Back-test: the model retrodicted last year 31% high. Findings shipped severity-ranked; the rebuilt v2 (assumptions sourced, plugs removed, checks visible, growth case honest + a labeled upside scenario) survived the investor's own diligence pass — their analyst's first three questions were the check-cell dashboard, the growth source, and the back-test, in that order. The four-quarter override, had it shipped, would have been found by that analyst instead.

## Related skills
- `budget-variance-analysis` — reality's ongoing feedback to any model.
- `unit-economics-model` — the assumption layer models most often inflate.
- `cash-flow-forecast` — the model type where step 5's stress matters most.
- `source-credibility-check` — auditing the external inputs.
