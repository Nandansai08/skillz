---
name: financial-model-review
description: >
  Use when auditing a spreadsheet financial model — hunting hardcodes,
  broken links, circular logic, and heroic assumptions; testing whether
  outputs survive sensitivity. Triggers: "review this model", "check the
  spreadsheet before the board sees it", "does this model hold up",
  "stress test these projections", "the numbers feel too good". NOT for
  building the model — this is the adversarial pass.
---

# Financial Model Review

## Overview

Models lie two ways: mechanically (the SUM stopping at row 40, the plug typed to make Q3 tie out) and rhetorically (growth at 2.7× trailing actuals, justified nowhere). The review hunts both — and its decision-grade output is the sensitivity pass: which single assumption flips the verdict, and does the model know?

## When to Use

- A model (budget, forecast, deal case, fundraise engine) is about to drive a real decision.
- Two models of the same business disagree and someone must arbitrate.

**When NOT to use:**
- Building the model — this is the adversarial pass on one that exists.

## Prerequisites

- The live model file (not a PDF — you audit formulas, not printouts), its purpose, and which outputs feed the decision.

## The Workflow

1. **Map the architecture before touching any number:** inputs → calculations → outputs — separated, or lasagna? Trace the 2–3 decision-driving outputs backward to their assumption roots; the paths you walk are the audit's spine. A model whose outputs can't be traced in an hour fails at the architecture layer regardless of its arithmetic.

2. **Hunt hardcodes — the spreadsheet's most reliable pathology:** numbers typed inside formulas (`=B4*1.2*0.85` — what is 0.85, who approved it?); the mechanical sweep plus the visual scan for the classic tell — a formula row with one hand-overridden cell mid-range (the plug typed to make a period tie out, silently disconnecting it from every assumption). Every hardcode: promoted to a labeled assumption or explained; unexplained plugs are findings. Three plugs clustering in one section means the section's LOGIC is wrong, not that three cells need promoting.

3. **Check structural integrity:**
   - **Circularity:** intentional (interest-on-average-debt, documented with iteration settings) vs accidental (a loop silenced by toggling iterative calc — a correctness bug in a settings band-aid).
   - **Links and ranges:** broken references, desktop-file links, ranges that didn't extend (the SUM stopping at row 40 of a 60-row list — spreadsheet-land's silent classic).
   - **Time consistency:** one period convention, no calendar/fiscal splices.
   - **Balance checks:** the balance sheet balances BY FORMULA with a visible check cell; cash flows tie to cash. A model without check cells expects its reader to supply the paranoia it lacks.

4. **Audit the assumptions against evidence — where models actually lie:** each key input: source? (actuals, market data, or wish — `source-credibility-check` for the external ones); consistent with history? (growth at 2.7× trailing actuals owes an explanation IN the model); internally consistent? (revenue tripling on flat headcount and constant CAC — the assumptions contradict each other; growth costs something). The hockey stick isn't automatically wrong — the undischarged burden of explaining it is.

5. **Run the sensitivity pass — the review's decision-grade output:** flex the 3–5 assumptions the step-1 trace showed carry the outputs (±20%, and to plausible-worst from history): which single assumption flips the decision? Does the model break mechanically under stress (negative cash the formulas ignore, implied >100% market share)? A verdict that dies at −10% on one input is a bet on that input — the review's job is saying so in exactly those words. And the assumption too scary to flex is the review's headline, not its exclusion.

6. **Reconcile against reality anchors:** back-test — feed last year's inputs, compare outputs to what happened (a model that can't retrodict shouldn't predict); tie current-period figures to accounting actuals; sanity-scale outputs (implied market share, revenue per employee vs comparables — the outside view that catches what cell-auditing can't).

7. **Report by severity with fix-paths, and leave the model better:** blockers (errors changing the decision) / material (hardcodes on load-bearing paths, missing sensitivity) / hygiene. Leave behind: a check-cell dashboard and version discipline ("board_v3_FINAL_v2 (1).xlsx" is a filename and a warning). The review that only produces a memo fixed nothing; check cells keep auditing after you've left.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "The outputs look reasonable — the model's probably fine" | Plausible and correct are independent: the example's revenue was understated $400k by a range bug and 31% overstated by an override — netting to 'reasonable.' Mechanics first, always. |
| "That 0.85 is just a standard haircut everyone uses" | Then label it as an assumption with its source. Unlabeled in-formula constants are invisible to sensitivity, review, and the next owner — the definition of a buried decision. |
| "The plug makes Q3 tie to actuals — that's calibration" | It silently disconnects Q3 from every assumption; the model now agrees with the past by fiat and predicts nothing. Plugs are findings, and clustered plugs indict the section's logic. |
| "Flexing retention that far is unrealistic — skip it" | The assumption too scary to flex is precisely the one the decision rides on. Refusing the flex doesn't make the exposure smaller; it makes it invisible (step 5's headline rule). |
| "The founder knows the growth rate is achievable" | Conviction isn't a source. 2.7× trailing actuals gets explained in the model or discounted by the first analyst who checks — as the example's was about to be. |
| "Softening the findings keeps the relationship workable" | The board then decides on the unflagged blocker, and the relationship meets the consequence anyway. Severity-honest or the review is complicity. |

## Red Flags

- Review conducted on a PDF of the model.
- Formulas with embedded constants on decision-driving paths.
- Hand-overridden cells mid-formula-range.
- No check cells anywhere; iterative calc on with nobody knowing why.
- Growth assumptions unexplained against trailing actuals.
- No back-test; no sensitivity table; version chaos in the filename.

## Verification

- [ ] Output-to-assumption traces documented for the decision-driving outputs.
- [ ] Hardcode sweep results: count, dispositions (promoted/explained), plugs flagged — list attached.
- [ ] Structural checks passed: ranges, links, time, balance check-cells — noted per item.
- [ ] Assumption audit table: source + history-consistency + internal-consistency per key input.
- [ ] Sensitivity run: the verdict-flipping assumption named; mechanical breaks under stress noted.
- [ ] Back-test and actuals-tie results attached.
- [ ] Check-cell dashboard + versioning left in the model — shown.

## Example

Fundraise model headed to a term-sheet negotiation; founder asks for a hostile read. Architecture map: assumptions reasonably separated, BUT the revenue engine traced back to a "Pipeline" tab last reconciled two quarters ago. Hardcode sweep: 23 constants-in-formulas; 3 material (a typed 0.92 "margin uplift" nobody could source; two Q3 plugs), 1 blocker — a growth formula overridden for exactly the four quarters shown in the deck's chart. Structural pass: one broken range (new products below a SUM's reach — revenue understated $400k, ironically favorable) and no check cells. Assumption audit: month-over-month growth at 2.7× trailing actuals, justified nowhere; sensitivity showed the raise's runway math flipped at −15% on that single input. Back-test: retrodicted last year 31% high. Findings severity-ranked; the rebuilt v2 (sourced assumptions, plugs removed, checks visible, honest growth case + labeled upside) survived the investor's own diligence — their analyst's first three questions were the check-cell dashboard, the growth source, and the back-test, in that order.

## Related skills

- `budget-variance-analysis` — reality's ongoing feedback to any model.
- `unit-economics-model` — the assumption layer models most often inflate.
- `cash-flow-forecast` — the model type where stress matters most.
- `source-credibility-check` — auditing the external inputs.
