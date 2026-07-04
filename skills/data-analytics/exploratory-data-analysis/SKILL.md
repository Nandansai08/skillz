---
name: exploratory-data-analysis
description: >
  Use when meeting a new dataset — a structured first pass through grain,
  nulls, distributions, outliers, and relationships before any modeling
  or reporting. Triggers: "explore this dataset", "EDA on", "what's in
  this data", "get familiar with this table", "profile this data", "first
  look at the data". NOT for fixing the problems found (see
  data-cleaning-pipeline) — EDA produces its worklist.
---

# Exploratory Data Analysis

## Overview

Every downstream number inherits EDA's honesty: get the grain wrong and revenue double-counts; miss the sentinel values and averages lie. The structured pass — grain, columns, distributions, time, relationships, outliers — ends in written findings, not forty plots.

## When to Use

- First contact with any dataset you'll analyze, model, or report on.
- Sanity-checking a familiar dataset after a pipeline change.

**When NOT to use:**
- Executing the fixes — `data-cleaning-pipeline` consumes this skill's worklist.

## Prerequisites

- The data loadable (or queryable — warehouse EDA via SQL follows the same recipe).
- Whatever documentation/schema exists, plus one question: what is one row supposed to represent?

## The Workflow

1. **Establish the grain first — what is one row?** One order? One order *item*? One status-change per order? Verify, don't trust the table name:
   ```python
   df.shape
   df['order_id'].is_unique          # if this should be the key and isn't →
   df[df.duplicated('order_id', keep=False)].sort_values('order_id').head(10)
   ```
   Duplicate "keys" usually mean the grain is finer than advertised (versioned rows, upstream join fan-out). Every later number is wrong until the grain is right.

2. **Profile columns mechanically:** `df.info()`, `df.describe(include='all')`, and per-column: dtype vs semantic type (dates as strings, IDs as floats — the `.0` suffix tell), null rate, distinct count. Distinct-count reading: n_distinct == n_rows on a non-key → free text; tiny n_distinct on a numeric → actually categorical; and `value_counts(dropna=False)` on categoricals — the `"N/A"`, `"unknown"`, `-999`, and `""` crowd are nulls wearing costumes.

3. **Distributions before statistics.** Histogram every numeric that matters, because the *shape* determines which statistics mean anything: heavy right-skew (revenue, latency — median and percentiles, never mean alone), spikes (a mass at exactly 0 or 9999 is a code, not a value), truncation at round numbers (collection limits), bimodality (two populations — find the splitting variable before aggregating them into one lie).

4. **Time is its own audit:** min/max of every timestamp (rows from 1970 or 2099 = epoch bugs), row counts per day/week plotted — gaps are pipeline outages, spikes are backfills or dupes; timezone consistency; and whether "created_at" means event time or load time. Data fine in aggregate is routinely missing three specific weeks. If sampling for speed, check the sampling: `LIMIT 10000` on a date-clustered table profiles January only.

5. **Relationships and integrity across columns:** correlations for numerics (screening, not conclusions), crosstabs for key categoricals, referential checks against related tables (orphaned foreign keys, categories in fact not in dim). Test the invariants the domain implies: `line_total ≈ qty × unit_price`? `ship_date ≥ order_date`? Violations counted, not assumed away — each is either a data bug or a domain rule you didn't know (both valuable).

6. **Outliers: locate, characterize, don't delete.** `df.nlargest(20, col)` and eyeball whole rows, not just values. Ask which kind: data error (negative age), unit mixup (bimodal ×3.28 = meters/feet), test/internal accounts (`email LIKE '%@yourco%'`), or real-and-important (the whale customer that IS the revenue story). Each kind gets different downstream treatment; that decision list is an EDA deliverable.

7. **Write the two outputs.** (a) A findings note: grain, row count, date coverage, null landscape, top 5 surprises with evidence, and the unresolved questions for the data owner. (b) The cleaning worklist for `data-cleaning-pipeline`. Ten bullets beat forty plots — the notebook is the appendix, not the deliverable.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "The table's called `orders` — one row per order, obviously" | The example's `subscriptions` table held 3.2 status-change rows per subscription, and the planned churn rate would have triple-counted. Table names are aspirations; `is_unique` is a fact. |
| "describe() gives me the summary — that's the profile" | Mean 45, median 4, and a spike at 9999 all hide inside the same describe() row. Shapes decide which statistics mean anything; only histograms show shapes. |
| "I'll drop the outliers to clean things up" | Some outliers ARE the story (the enterprise whale), some are unit bugs, some are test accounts — deleting before characterizing throws away signal and hides defects alike. |
| "Nulls are obvious — pandas shows them" | `-999`, `""`, `"N/A"`, and `1900-01-01` don't show as nulls; they show as data, and they average right into your means. value_counts is the detector. |
| "The deadline is tight — profile later, analyze now" | The grain bug found in EDA costs an hour; found in the exec meeting it costs the analysis's credibility and a redo. EDA is the fast path, not the detour. |
| "The data team already validated this table" | They validated their contract, not your question's assumptions. The invariant checks (step 5) test what YOUR analysis is about to lean on. |

## Red Flags

- Analysis code written before any uniqueness/grain check ran.
- No histogram exists for the metric being reported on.
- Time-coverage never plotted; the missing weeks discovered by a stakeholder.
- Sentinel values visible in value_counts, still present in aggregates.
- An EDA "deliverable" that is a notebook of plots with no written findings.
- Zero questions sent to the data owner (real datasets always generate some).

## Verification

- [ ] Grain verified with the uniqueness check — output in the findings note.
- [ ] Per-column profile done; sentinel/costume-null inventory listed.
- [ ] Distributions plotted for all analysis-relevant numerics — shapes noted.
- [ ] Time coverage plotted; gaps/spikes explained or flagged.
- [ ] Domain invariants tested with violation counts — listed.
- [ ] Findings note (≤1 page) + cleaning worklist delivered — links; owner questions sent.

## Example

New dataset: `subscriptions` table for churn analysis. Step 1: `subscription_id` not unique — grain is subscription-*status-change* (avg 3.2 rows each); the planned churn rate would have triple-counted. Step 2: `plan_type` value_counts shows 5 expected values plus `""` (4%) and `legacy_gold` (0.2%, undocumented). Step 4: daily counts show zero rows for two weeks in March — confirmed pipeline outage, must be footnoted in any trend. Step 5: 3% of `user_id`s missing from `users` (deleted accounts, kept in facts). Step 6: 40 subscriptions with $0 price — internal test accounts by email domain. Findings note: 9 bullets, 4 questions to the data owner (two answered with "oh right, that migration"), cleaning worklist of 6 items. The churn analysis started a day later and survived review — because the grain bug died in EDA instead of in the meeting.

## Related skills

- `data-cleaning-pipeline` — executing the worklist this produces.
- `metric-definition` — defining measures on the now-understood grain.
- `cohort-retention-analysis` — a downstream analysis that inherits these checks.
