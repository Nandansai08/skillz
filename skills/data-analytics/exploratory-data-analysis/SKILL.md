---
name: exploratory-data-analysis
description: >
  Use when meeting a new dataset — a structured first pass through shape,
  nulls, distributions, outliers, and relationships before any modeling or
  reporting. Triggers: "explore this dataset", "EDA on", "what's in this
  data", "get familiar with this table", "profile this data",
  "first look at the data".
---

# Exploratory Data Analysis

## When to use this skill
- First contact with any dataset you'll analyze, model, or report on.
- Sanity-checking a familiar dataset after a pipeline change.
- NOT for cleaning the problems you find — that's `data-cleaning-pipeline`; EDA produces its worklist.

## Prerequisites
- The data loadable (or queryable — EDA on a warehouse table via SQL is the same recipe).
- Whatever documentation/schema exists, plus one question: what is one row supposed to represent?

## Workflow

1. **Establish the grain first — what is one row?** One order? One order *item*? One order per day snapshot? Verify, don't trust the table name:
   ```python
   df.shape
   df['order_id'].is_unique          # if this should be the key and isn't →
   df[df.duplicated('order_id', keep=False)].sort_values('order_id').head(10)
   ```
   Duplicate "keys" usually mean the grain is finer than advertised (versioned rows, joins upstream fanning out). Every later number is wrong until the grain is right.

2. **Profile columns mechanically:** `df.info()`, `df.describe(include='all')`, and per-column: dtype vs semantic type (dates stored as strings, IDs as floats — the `.0` suffix tell), null rate, distinct count. Distinct-count reading: n_distinct == n_rows on a non-key → free text; tiny n_distinct on a numeric → it's actually categorical; unexpected values in categoricals (`df[col].value_counts(dropna=False).head(20)`) — the `"N/A"`, `"unknown"`, `-999`, and `""` crowd are nulls wearing costumes.

3. **Distributions before statistics.** Histogram every numeric that matters (`df[col].hist(bins=50)`), because the *shape* determines which statistics mean anything: heavy right-skew (revenue, latency — median and percentiles, never mean alone), spikes (a mass at exactly 0 or 9999 is a code, not a value), truncation/clipping at round numbers (collection limits), bimodality (two populations — find the splitting variable before aggregating them into one lie).

4. **Time is its own audit:** min/max of every timestamp (rows from 1970 or 2099 = epoch bugs), row counts per day/week plotted (`df.set_index('ts').resample('D').size().plot()`) — gaps are pipeline outages, spikes are backfills or dupes; check timezone consistency and whether "created_at" means event time or load time. Data that looks fine in aggregate is routinely missing three specific weeks.

5. **Relationships and integrity across columns:** correlations for numerics (`df.corr(numeric_only=True)` — screening, not conclusions), crosstabs for key categoricals, and referential checks against related tables (orphaned foreign keys, categories in fact not in dim). Test the invariants the domain implies: `line_total ≈ qty × unit_price`? `ship_date ≥ order_date`? Violations counted, not assumed away — each is either a data bug or a domain rule you didn't know (both valuable).

6. **Outliers: locate, characterize, don't delete.** `df.nlargest(20, col)` and eyeball whole rows, not just the value. Ask which kind: data error (negative age), unit mixup (meters vs feet — bimodal ×3.28), test/internal accounts (`email LIKE '%@yourco%'`), or real-and-important (the whale customer that IS the revenue story). Each kind gets different treatment downstream; that decision list is an EDA deliverable.

7. **Write the two outputs.** (a) A findings note: grain, row count, date coverage, null landscape, the top 5 surprises with evidence, and the "unresolved questions for the data owner" list. (b) The cleaning worklist for `data-cleaning-pipeline`. Ten bullets beat forty plots — the notebook is the appendix, not the deliverable.

## Common pitfalls
- Skipping the grain check and computing revenue on a table with versioned rows — everything double-counts and the dashboard ships before anyone notices. Step 1 is not optional.
- Trusting `describe()` without histograms: mean 45, median 4, and a spike at 9999 all hide inside the same summary row.
- Sentinel values averaged in: `-999` "unknowns" pulling the mean negative; `1900-01-01` birthdays making users 126 years old. Step 2's value_counts is the detector.
- Dropping outliers reflexively "to clean the data" — deleting the enterprise customers and then reporting that revenue is stable.
- EDA that's 40 plots and no sentences. If it doesn't end in written findings and questions, it was tourism, not analysis.
- Profiling a sample without checking the sampling: `LIMIT 10000` on a table clustered by date = you profiled January.

## Example
New dataset: `subscriptions` table for churn analysis. Step 1: `subscription_id` not unique — grain is subscription-*status-change* (avg 3.2 rows each); the planned churn rate would have triple-counted. Step 2: `plan_type` value_counts shows 5 expected values plus `""` (4%) and `legacy_gold` (0.2%, undocumented). Step 4: daily counts show zero rows for two weeks in March — confirmed pipeline outage, must be footnoted in any trend. Step 5: 3% of `user_id`s missing from `users` (deleted accounts, kept in facts). Step 6: 40 subscriptions with $0 price — internal test accounts by email domain. Findings note: 9 bullets, 4 questions to the data owner (two answered with "oh right, that migration"), cleaning worklist of 6 items. The churn analysis started a day later and survived review — because the grain bug died in EDA instead of in the meeting.

## Related skills
- `data-cleaning-pipeline` — executing the worklist this produces.
- `metric-definition` — defining measures on the now-understood grain.
- `cohort-retention-analysis` — a downstream analysis that inherits these checks.
