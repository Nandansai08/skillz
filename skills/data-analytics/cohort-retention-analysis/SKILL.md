---
name: cohort-retention-analysis
description: >
  Use when measuring whether users/customers stick around — building cohort
  tables, reading retention curves, avoiding the classic misreads (mixed
  cohorts, survivorship, calendar artifacts). Triggers: "retention analysis",
  "cohort table", "churn by cohort", "are users coming back", "is retention
  improving", "D7/D30 retention".
---

# Cohort Retention Analysis

## When to use this skill
- Answering "is the product getting stickier?" or "did feature/campaign X improve retention?"
- Building or auditing a retention dashboard.
- NOT for defining what "active" means — that's `metric-definition`, and it's the mandatory first stop (a retention curve on a mushy activity definition is precise nonsense).

## Prerequisites
- Event data with user ID + timestamp, and each user's cohort anchor (signup date, first purchase, first use of the feature).
- The activity definition, pinned per `metric-definition` step 1.

## Workflow

1. **Fix the three definitions before any SQL:** the **cohort anchor** (signup? first *meaningful* action? — for products with delayed activation, anchoring on signup mixes tourists into every cohort), the **activity event** (the thing that indicates real use — not logins), and the **period convention**: day-N (calendar days since anchor), unbounded/rolling ("active on day ≥ N"), or bounded ("active *in* week N") — they produce different numbers from identical data, and mixing them across a dashboard is the classic apples-to-oranges bug. Write all three at the top of the analysis.

2. **Build the standard cohort matrix** — rows = cohort (by anchor week/month), columns = periods since anchor, cells = % of cohort active:
   ```sql
   SELECT cohort_week,
          FLOOR(DATEDIFF('day', anchor_date, activity_date) / 7) AS week_n,
          COUNT(DISTINCT user_id)::float / MAX(cohort_size) AS retention
   FROM activity JOIN cohorts USING (user_id)
   GROUP BY 1, 2
   ```
   Two structural rules: cells where the cohort hasn't lived long enough are **blank, not zero** (the bottom-right triangle — plotting immature cells as churn is the most common chart lie in this genre); and the denominator is the *original* cohort size, always.

3. **Read the matrix in two directions, separately:**
   - **Across a row** (one cohort over time): the curve's shape. The healthy signature is a drop then a **flatten** — a plateau means a retained core exists; a curve that decays toward zero without flattening means no product-market fit for that cohort, whatever the D7 number says.
   - **Down a column** (same-age comparison across cohorts): is week-4 retention improving for newer cohorts? This is the *only* valid way to ask "is retention improving" — comparing cohorts at the same age. The aggregate "monthly retention rate" mixes cohort ages and moves when growth accelerates, not when the product improves.

4. **Hunt the composition confound before crediting any change.** New cohorts differ in *who they are*, not just what they experienced: a marketing push fills a cohort with low-intent users (retention "drops" though the product is unchanged); a price increase filters for high-intent (retention "improves"). Segment the matrix by acquisition channel/plan/geo before attributing column-wise movement to product changes — and treat any cohort coinciding with a campaign as suspect until segmented.

5. **Check the calendar artifacts:** day-N retention oscillates with a 7-day period for weekday-used products (D6 vs D7 crossing a weekend); holidays dent every cohort's same *calendar* week, which shows up as a diagonal stripe in the matrix — a diagonal pattern means calendar effect, a horizontal pattern means cohort effect, a vertical pattern means age effect. Learning to read the stripes is half the skill.

6. **Quantify beyond the eyeball when it matters:** compare cohort curves with confidence intervals on the cell proportions (small cohorts wobble — a 200-user cohort's ±7% swamps most real effects); summarize a curve as its plateau height and time-to-plateau rather than one arbitrary D-number; for revenue retention, compute NRR/GRR on the same cohort discipline (expansion hides churn in blended NRR — report both).

7. **Turn the finding into a mechanism hypothesis and test it.** "Cohorts that used feature X in week 1 retain 2× better" is a *correlation* — high-intent users do more of everything (the classic "power users eat breakfast" fallacy). The activation candidates this analysis surfaces go to `ab-test-analysis` for causal confirmation before they become the onboarding roadmap. The cohort analysis finds the candidates; the experiment convicts them.

## Common pitfalls
- Averaging retention across cohorts of different ages ("our retention is 34%") — the number changes with growth rate alone. Same-age columns or nothing.
- Plotting the immature triangle as zeros: the dashboard shows every recent cohort "collapsing" and someone declares an emergency. Blank ≠ churned.
- Survivorship framing: analyzing "users who churned" by their last-month behavior finds that disengagement precedes churn — a tautology dressed as insight. Cohort-forward analysis, not churn-backward.
- Activity defined as login/session while the product's value event is something else — retention of *token refreshes* improves while actual usage dies.
- Ignoring cohort size when comparing: the January cohort (n=12,000) vs the launch-week cohort (n=340) plotted as equals; the wobble is noise.
- Shipping the correlation from step 7 straight into onboarding ("make everyone do X!") without the experiment — X was a symptom of intent, the forced version moves nothing, a quarter is spent.

## Example
Question: "did the new onboarding (shipped week 10) improve retention?" Matrix built: anchor = first project created (not signup — 40% of signups never create one and were polluting cohorts), activity = weekly project edit, bounded weeks. Column read: week-4 retention for cohorts 11–14 = 31–33% vs 24–26% for cohorts 6–9 — promising. Confound check: week 11 coincided with a paid campaign; segmented by channel, the organic-only comparison still showed +5pts (26%→31%) — the improvement survives composition control. Diagonal check: one stripe from a Black Friday week, noted, ignored. Curve shape: plateau rose from ~22% to ~27%, time-to-plateau unchanged (~week 5). Verdict shipped with appropriate humility: "consistent with a real improvement, +4–6pts at plateau, organic-controlled; causal confirmation via holdback experiment recommended" — the holdback ran and confirmed +4pts. The dashboard version got blank-triangle handling and a channel filter as permanent features.

## Related skills
- `metric-definition` — the activity/anchor definitions this analysis stands on.
- `ab-test-analysis` — converting step 7's correlations into causes.
- `exploratory-data-analysis` — the data-trust groundwork underneath.
