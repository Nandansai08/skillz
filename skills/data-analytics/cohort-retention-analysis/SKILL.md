---
name: cohort-retention-analysis
description: >
  Use when measuring whether users/customers stick around — building
  cohort tables, reading retention curves, avoiding the classic misreads
  (mixed cohorts, survivorship, calendar artifacts). Triggers: "retention
  analysis", "cohort table", "churn by cohort", "are users coming back",
  "is retention improving", "D7/D30 retention". NOT for defining what
  "active" means (see metric-definition — the mandatory first stop).
---

# Cohort Retention Analysis

## Overview

"Is retention improving?" is only answerable by comparing cohorts at the same age — every aggregate retention number moves with growth rate alone. The matrix, its blank-not-zero triangle, and the composition-confound check are what separate the analysis from the artifact.

## When to Use

- Answering "is the product getting stickier?" or "did feature/campaign X improve retention?"
- Building or auditing a retention dashboard.

**When NOT to use:**
- Defining the activity/anchor events — `metric-definition` first; a retention curve on a mushy definition is precise nonsense.

## Prerequisites

- Event data with user ID + timestamp, and each user's cohort anchor (signup, first purchase, first use).
- The activity definition, pinned per `metric-definition` step 1.

## The Workflow

1. **Fix the three definitions before any SQL:** the **cohort anchor** (signup? first *meaningful* action? — for products with delayed activation, anchoring on signup mixes tourists into every cohort), the **activity event** (the thing indicating real use — not logins), and the **period convention**: day-N, unbounded ("active on day ≥ N"), or bounded ("active *in* week N") — they produce different numbers from identical data, and mixing them across a dashboard is the classic apples-to-oranges bug.

2. **Build the standard cohort matrix** — rows = cohort (by anchor week/month), columns = periods since anchor, cells = % of cohort active:
   ```sql
   SELECT cohort_week,
          FLOOR(DATEDIFF('day', anchor_date, activity_date) / 7) AS week_n,
          COUNT(DISTINCT user_id)::float / MAX(cohort_size) AS retention
   FROM activity JOIN cohorts USING (user_id)
   GROUP BY 1, 2
   ```
   Two structural rules: cells where the cohort hasn't lived long enough are **blank, not zero** (plotting immature cells as churn is the most common chart lie in this genre); and the denominator is the *original* cohort size, always.

3. **Read the matrix in two directions, separately:**
   - **Across a row** (one cohort over time): the curve's shape. The healthy signature is a drop then a **flatten** — a plateau means a retained core exists; decay-to-zero without flattening means no fit for that cohort, whatever the D7 number says.
   - **Down a column** (same-age comparison across cohorts): the *only* valid way to ask "is retention improving." The aggregate "monthly retention rate" mixes cohort ages and moves when growth accelerates, not when the product improves.

4. **Hunt the composition confound before crediting any change.** New cohorts differ in *who they are*: a marketing push fills a cohort with low-intent users (retention "drops" though the product is unchanged); a price increase filters for high-intent ("improves"). Segment the matrix by acquisition channel/plan/geo before attributing column-wise movement to product changes — any cohort coinciding with a campaign is suspect until segmented.

5. **Check the calendar artifacts:** day-N retention oscillates with a 7-day period for weekday products; holidays dent every cohort's same *calendar* week. The stripe-reading rule: a **diagonal** pattern in the matrix = calendar effect, **horizontal** = cohort effect, **vertical** = age effect. Learning to read the stripes is half the skill.

6. **Quantify beyond the eyeball when it matters:** confidence intervals on cell proportions (a 200-user cohort's ±7% swamps most real effects — never compare a 12,000-user cohort against a 340-user one as equals); summarize a curve as plateau height + time-to-plateau rather than one arbitrary D-number; for revenue retention, NRR/GRR on the same cohort discipline (expansion hides churn in blended NRR — report both).

7. **Turn the finding into a mechanism hypothesis and test it.** "Cohorts that used feature X in week 1 retain 2× better" is a *correlation* — high-intent users do more of everything (the "power users eat breakfast" fallacy). The activation candidates this analysis surfaces go to `ab-test-analysis` for causal confirmation before they become the onboarding roadmap. Cohort analysis finds the candidates; the experiment convicts them.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "Our overall retention rate is 34% — that's the health number" | That number moves with growth rate alone: accelerate signups and it 'drops' with zero product change. Same-age columns or the number is measuring your marketing calendar. |
| "Recent cohorts are collapsing — look at the right side of the chart" | The bottom-right triangle is immature, not churned. Blank-not-zero (step 2) exists because this exact misread generates a fake emergency per quarter. |
| "Retention jumped after the campaign — the product changes worked" | The campaign changed WHO signed up, not just how many. Un-segmented column movement attributes composition shifts to product; the channel split (step 4) is mandatory before credit. |
| "Users who do X retain better — make everyone do X" | High-intent users do more of everything; X is a symptom of intent until an experiment says otherwise. The forced version of X moves nothing, a quarter is spent (step 7's conviction rule). |
| "Churned-user analysis shows disengagement precedes churn" | That's a tautology dressed as insight — of course leaving users disengage first. Cohort-forward analysis, not churn-backward archaeology. |
| "D7 is the industry-standard number — just report that" | One arbitrary cell can't show whether the curve flattens — and the plateau is the entire product-market-fit signal. Curve shape first; D-numbers as summaries after. |

## Red Flags

- A retention "trend" chart built on the blended aggregate.
- Immature cells rendered as zeros in any dashboard.
- Cohort comparisons across a campaign boundary with no channel segmentation.
- Tiny and huge cohorts plotted as equal lines, no intervals.
- An "activation insight" heading straight to roadmap without an experiment.
- Activity defined as login while the value event is something else.

## Verification

- [ ] The three definitions (anchor, activity, period convention) written and consistent across the analysis — stated up top.
- [ ] Matrix built with original-size denominators and blank immature cells — spot-check the triangle.
- [ ] "Improving?" answered only via same-age column comparisons — the reading shown.
- [ ] Composition check done: segmented by channel/plan for any period with acquisition changes — the segmented view attached.
- [ ] Stripe scan performed (diagonal/horizontal/vertical) — artifacts noted.
- [ ] Correlational findings labeled as hypotheses with their planned experiment — linked.

## Example

Question: "did the new onboarding (shipped week 10) improve retention?" Matrix built: anchor = first project created (not signup — 40% of signups never create one and were polluting cohorts), activity = weekly project edit, bounded weeks. Column read: week-4 retention for cohorts 11–14 = 31–33% vs 24–26% for cohorts 6–9 — promising. Confound check: week 11 coincided with a paid campaign; segmented by channel, the organic-only comparison still showed +5pts (26%→31%) — the improvement survives composition control. Diagonal check: one stripe from a Black Friday week, noted, ignored. Curve shape: plateau rose from ~22% to ~27%, time-to-plateau unchanged (~week 5). Verdict shipped with appropriate humility: "consistent with a real improvement, +4–6pts at plateau, organic-controlled; causal confirmation via holdback experiment recommended" — the holdback ran and confirmed +4pts. The dashboard version got blank-triangle handling and a channel filter as permanent features.

## Related skills

- `metric-definition` — the activity/anchor definitions this analysis stands on.
- `ab-test-analysis` — converting step 7's correlations into causes.
- `exploratory-data-analysis` — the data-trust groundwork underneath.
