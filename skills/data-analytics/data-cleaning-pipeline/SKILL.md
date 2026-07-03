---
name: data-cleaning-pipeline
description: >
  Use when turning messy data into analysis-ready data reproducibly — dedupe,
  normalization, type fixing, sentinel handling — as scripted steps, not
  hand edits. Triggers: "clean this data", "dedupe these records",
  "messy CSV", "standardize this dataset", "fix the data quality",
  "prepare data for analysis".
---

# Data Cleaning Pipeline

## When to use this skill
- EDA produced a worklist of data problems and analysis is blocked on them.
- A recurring feed arrives dirty and needs a repeatable cleaning stage.
- NOT for exploring what's wrong in the first place — run `exploratory-data-analysis` first; cleaning without profiling is guessing.

## Prerequisites
- The EDA findings (grain, null landscape, sentinels, outlier decisions).
- Raw data kept immutable somewhere — cleaning always writes a NEW artifact.

## Workflow

1. **Script everything; never hand-edit the data.** The pipeline is code (SQL models, a Python script, dbt staging models) that reads raw → writes clean. Hand-edited spreadsheets can't be re-run when next month's file arrives, can't be reviewed, and can't be trusted. Raw stays raw forever (this is the bronze-layer instinct — `medallion-architecture-design`).

2. **Order the stages so each works on the previous one's guarantees:**
   type coercion → structural fixes (grain/reshaping) → value normalization → sentinel/null handling → dedupe → validation. Dedupe before types are fixed misses `"42"` vs `42` duplicates; normalize before dedupe or `"ACME Corp"` and `"acme corp "` stay two customers.

3. **Type coercion with a loud reject lane.** Parse dates with explicit formats (never silent `dayfirst` guessing on ambiguous data), numerics via `pd.to_numeric(errors='coerce')` **followed by counting the new NaNs** — coercion that silently nulls 8% of a column is data loss wearing a lab coat. Rows that fail coercion go to a rejects file with reasons, not into the void:
   ```python
   df['amount_clean'] = pd.to_numeric(df['amount'], errors='coerce')
   rejects = df[df['amount'].notna() & df['amount_clean'].isna()]
   ```

4. **Normalize values against explicit mappings, not clever fuzz.** Case/whitespace/unicode-NFC first; then categorical unification via a *reviewable mapping table* (`"US", "USA", "United States" → "US"`) checked into the repo. Fuzzy matching (rapidfuzz) only proposes candidates for human review of the mapping — auto-merging on string similarity is how "Johnson Ltd" and "Johnsen Ltd" (different companies) become one.

5. **Handle sentinels and nulls per-column, per the EDA decisions:** `-999`/`9999`/`""`/`"N/A"` → real NULL; then each column gets a documented policy — leave null (usually right for analysis), impute (only with a recorded method and an `_imputed` flag column if models need it), or drop rows (only when the column is essential and the loss is counted). "Fill with 0" is a policy with consequences (0 is a *value*); choose it consciously or not at all.

6. **Dedupe with a defined survivor rule.** Define the match key (post-normalization), then the survivorship: most recent by `updated_at`? Most complete (fewest nulls)? Source-priority? `drop_duplicates(keep='first')` without a deliberate sort is "keep a random one." Log the dedupe count and keep the losers queryable (a `dedupe_removed` output) — the day someone asks "where did order 4412 go," you answer in one query.

7. **Close with validation gates and a cleaning report.** Assertions the clean data must pass — schema (types, required columns), invariants from EDA (totals ≥ 0, ship ≥ order date), row-count reconciliation (`raw = clean + rejects + deduped`, to the row), and drift checks for recurring feeds (today's null rate within tolerance of history — Great Expectations/pandera/dbt tests make these declarative). The report prints every mutation's count: rows in, coerced, rejected, normalized, deduped, out. That reconciliation line is the pipeline's integrity proof.

## Common pitfalls
- Cleaning destructively over the raw file. The question "wait, what did the original say?" arrives within a week, guaranteed.
- Silent coercion loss: `errors='coerce'` without the NaN-delta check is the single most common way real values vanish.
- Dropping every row with any null (`dropna()`) — losing 40% of the data and, worse, a *biased* 40% (nulls correlate with segments — old records, one source system).
- Deduping on raw strings pre-normalization, or on a "unique" key the EDA already proved non-unique at a different grain.
- Imputation without a flag: three months later a model treats 10,000 imputed medians as real observations and nobody remembers.
- A pipeline that runs once. If the feed recurs, wire the validation gates to fail loudly on next month's surprises — clean data rots (`data-pipeline-idempotency` for the rerun semantics).

## Example
Monthly vendor CSV, ~200k rows, EDA worklist: dates in two formats, `amount` with `$` and parentheses-negatives, vendor names free-text, `9999` as unknown-quantity, ~3% duplicate shipments. Pipeline built as six numbered functions matching the stage order; mapping table for 340 vendor variants (fuzzy-proposed, human-approved, in git). First run's report: 201,441 in → 198,204 clean, 1,882 rejects (unparseable dates from one legacy source — sent back to vendor with line numbers), 1,355 deduped (survivor = latest file date), reconciliation exact. Month 3: the drift gate failed — null rate on `amount` jumped 0.2% → 11%; vendor had changed the column name. Caught at load, not in the quarterly report.

## Related skills
- `exploratory-data-analysis` — produces this pipeline's worklist.
- `data-pipeline-idempotency` — making the recurring run rerun-safe.
- `medallion-architecture-design` — where these stages live in a lakehouse.
