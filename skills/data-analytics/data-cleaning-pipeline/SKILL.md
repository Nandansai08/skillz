---
name: data-cleaning-pipeline
description: >
  Use when turning messy data into analysis-ready data reproducibly —
  dedupe, normalization, type fixing, sentinel handling — as scripted
  stages, not hand edits. Triggers: "clean this data", "dedupe these
  records", "messy CSV", "standardize this dataset", "fix the data
  quality", "prepare data for analysis". NOT for discovering what's wrong
  in the first place — run exploratory-data-analysis first; cleaning
  without profiling is guessing.
---

# Data Cleaning Pipeline

## Overview

Cleaning is code: raw stays immutable, every mutation is a counted stage, rejects go to a lane instead of the void, and the run ends with a reconciliation that balances to the row. Hand-edited spreadsheets can't re-run next month, can't be reviewed, and can't be trusted.

## When to Use

- EDA produced a worklist of data problems and analysis is blocked on them.
- A recurring feed arrives dirty and needs a repeatable cleaning stage.

**When NOT to use:**
- Exploring what's wrong — `exploratory-data-analysis` first.

## Prerequisites

- The EDA findings (grain, null landscape, sentinels, outlier decisions).
- Raw data kept immutable somewhere — cleaning always writes a NEW artifact.

## The Workflow

1. **Script everything; never hand-edit the data.** The pipeline is code (SQL models, a Python script, dbt staging models) that reads raw → writes clean. Raw stays raw forever — the question "wait, what did the original say?" arrives within a week, guaranteed.

2. **Order the stages so each works on the previous one's guarantees:**
   type coercion → structural fixes (grain/reshaping) → value normalization → sentinel/null handling → dedupe → validation. Dedupe before types are fixed misses `"42"` vs `42` duplicates; normalize before dedupe or `"ACME Corp"` and `"acme corp "` stay two customers.

3. **Type coercion with a loud reject lane.** Parse dates with explicit formats (never silent `dayfirst` guessing), numerics via `pd.to_numeric(errors='coerce')` **followed by counting the new NaNs** — coercion that silently nulls 8% of a column is data loss wearing a lab coat. Rows that fail coercion go to a rejects file with reasons:
   ```python
   df['amount_clean'] = pd.to_numeric(df['amount'], errors='coerce')
   rejects = df[df['amount'].notna() & df['amount_clean'].isna()]
   ```

4. **Normalize values against explicit mappings, not clever fuzz.** Case/whitespace/unicode-NFC first; then categorical unification via a *reviewable mapping table* (`"US", "USA", "United States" → "US"`) checked into the repo. Fuzzy matching only proposes candidates for human review of the mapping — auto-merging on string similarity is how "Johnson Ltd" and "Johnsen Ltd" (different companies) become one.

5. **Handle sentinels and nulls per-column, per the EDA decisions:** `-999`/`9999`/`""`/`"N/A"` → real NULL; then each column gets a documented policy — leave null (usually right for analysis), impute (only with a recorded method and an `_imputed` flag column), or drop rows (only when the column is essential and the loss is counted). "Fill with 0" is a policy with consequences (0 is a *value*); choose it consciously or not at all.

6. **Dedupe with a defined survivor rule.** Match key (post-normalization), then survivorship: most recent by `updated_at`? Most complete? Source-priority? `drop_duplicates(keep='first')` without a deliberate sort is "keep a random one." Log the dedupe count and keep the losers queryable — the day someone asks "where did order 4412 go," you answer in one query.

7. **Close with validation gates and a cleaning report.** Assertions the clean data must pass — schema, invariants from EDA, row-count reconciliation (`raw = clean + rejects + deduped`, to the row), and drift checks for recurring feeds (today's null rate within tolerance of history — Great Expectations/pandera/dbt tests). The report prints every mutation's count: rows in, coerced, rejected, normalized, deduped, out. That reconciliation line is the pipeline's integrity proof.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "It's a one-off analysis — I'll just fix the file by hand" | The 'one-off' recurs next month, the hand-fixes are unrepeatable and unreviewable, and nobody can say what changed. Thirty minutes of script beats an untraceable artifact every time. |
| "errors='coerce' handles the bad values" | It handles them by silently deleting them. The NaN-delta count after every coercion is the difference between cleaning and quiet data loss. |
| "Fuzzy matching at 90% similarity is safe to auto-merge" | 'Johnson Ltd' and 'Johnsen Ltd' clear 90% and are different companies. Fuzz proposes; a human approves the mapping table — which then lives in git, reviewable. |
| "Drop rows with any null — clean dataset" | dropna() loses 40% of the data AND a biased 40% (nulls correlate with segments: old records, one source system). Per-column policy or the 'clean' dataset is a skewed one. |
| "Impute the median, models need complete data" | Unflagged imputation means three months later 10,000 medians are treated as observations. The `_imputed` flag column costs nothing; its absence costs the model's honesty. |
| "The totals look right — reconciliation is overkill" | 'Looks right' at 200k rows hides a thousand silently vanished ones. raw = clean + rejects + deduped, to the row, is four lines of code and the entire integrity proof. |

## Red Flags

- A "cleaned" file existing with no script that produces it.
- Coercion steps with no before/after null counts.
- Dedupe with no survivor rule stated (keep='first' on unsorted data).
- Sentinels handled for some columns, averaged into others.
- No rejects artifact — invalid rows just gone.
- Recurring feed with no drift gates; the schema change discovered in the quarterly report.

## Verification

- [ ] Pipeline is code in version control; raw immutable — repo link.
- [ ] Stage order follows the dependency chain (types→structure→normalize→sentinels→dedupe→validate) — visible in the code.
- [ ] Every coercion's NaN-delta counted and reported — report excerpt.
- [ ] Mapping tables reviewed and committed; no auto-merged fuzzy matches.
- [ ] Per-column null policy documented; imputations flagged in-data.
- [ ] Reconciliation exact: raw = clean + rejects + deduped — the report line quoted.
- [ ] Recurring feeds: drift gates live — test config linked.

## Example

Monthly vendor CSV, ~200k rows, EDA worklist: dates in two formats, `amount` with `$` and parentheses-negatives, vendor names free-text, `9999` as unknown-quantity, ~3% duplicate shipments. Pipeline built as six numbered functions matching the stage order; mapping table for 340 vendor variants (fuzzy-proposed, human-approved, in git). First run's report: 201,441 in → 198,204 clean, 1,882 rejects (unparseable dates from one legacy source — sent back to vendor with line numbers), 1,355 deduped (survivor = latest file date), reconciliation exact. Month 3: the drift gate failed — null rate on `amount` jumped 0.2% → 11%; vendor had changed the column name. Caught at load, not in the quarterly report.

## Related skills

- `exploratory-data-analysis` — produces this pipeline's worklist.
- `data-pipeline-idempotency` — making the recurring run rerun-safe.
- `medallion-architecture-design` — where these stages live in a lakehouse.
