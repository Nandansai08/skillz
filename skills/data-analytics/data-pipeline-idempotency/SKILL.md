---
name: data-pipeline-idempotency
description: >
  Use when making a data pipeline safe to re-run — upserts,
  overwrite-partition semantics, watermarks, backfills that don't
  double-count. Triggers: "pipeline reran and duplicated data", "make
  this job idempotent", "safe to backfill?", "watermark for incremental
  load", "rerun the DAG", "exactly-once processing". NOT for one-off
  exploratory transforms — though the moment a "one-off" gets a schedule,
  this applies.
---

# Data Pipeline Idempotency

## Overview

Every pipeline WILL be re-run — retries, backfills, operator panic — so "same logical input → same output state, any number of runs" is the property that separates a pipeline from a scheduled incident. Two write shapes deliver it: overwrite-by-partition and keyed merge; blind append delivers the double-count.

## When to Use

- Building or fixing any scheduled/triggered pipeline.
- A re-run just double-counted and you're cleaning up.

**When NOT to use:**
- Genuinely one-off transforms — until they get a schedule.

## Prerequisites

- The pipeline's write pattern (append? merge? overwrite?) and its natural key or partition scheme.

## The Workflow

1. **State the invariant: same logical input → same output state, no matter how many runs.** Test mentally (then actually): run the job twice for the same day — identical result to running once? If the answer involves "well, don't do that," the pipeline is a scheduled incident.

2. **Pick the write pattern by what the job produces — the two safe shapes:**
   - **Overwrite-by-partition** (the batch workhorse): the job owns a partition (usually a date) and replaces it wholesale — `INSERT OVERWRITE PARTITION (ds='2026-07-02')`, Delta `replaceWhere`. Re-run = same partition rewritten = idempotent by construction. Requires: partition key derivable from the run's logical date, and the job reads *all* input for that partition — overwrite semantics with incremental reads means the re-run overwrites a full partition with a slice of it.
   - **Merge/upsert on a natural key** (entities, CDC): `MERGE INTO ... ON target.id = source.id`, with a deterministic tie-break (latest `updated_at` wins) so replayed batches converge.
   Blind `INSERT`/append is the unsafe shape — every retry a duplicate. If append is forced (immutable event stores), dedupe downstream on a producer-supplied event ID — idempotency starts at the source.

3. **Separate logical time from wall-clock time.** The job for `2026-07-02` processes data *belonging to* that date, regardless of when it runs — the run date comes from the scheduler's logical/execution date, never `datetime.now()` or `CURRENT_DATE`. `now()`-based jobs produce different results on re-run by definition, and backfills become impossible.

4. **Incremental loads: watermark with a lag, tolerate overlap.** Track the high-water mark in a state table updated *transactionally with the load itself* (state advanced but load failed = skipped data; load done but state not = reprocessing — which is why step 2's merge matters: overlap must be harmless). Subtract a late-arrival lag (`WHERE updated_at > :wm - INTERVAL '1 hour'`) so slightly-late records get caught by the overlap-tolerant merge instead of lost.

5. **Make side effects idempotent too — the forgotten half.** The data write is idempotent but the job also sends emails, fires webhooks, bumps counters → the re-run spams customers. Options: idempotency keys on external calls, a sent-log checked before sending, or move side effects downstream of the *data* (which sees one state, not N runs). A pipeline is as idempotent as its worst side effect.

6. **Design backfills as first-class:** parameterized by date range, running the SAME code path as the daily run (a separate backfill script diverges within months and becomes the thing that double-counts), bounded parallelism (30 partition-overwrites at once is a warehouse self-DDoS — and cumulative metrics force sequential order), and a post-backfill reconciliation query (counts and totals per day vs source) as the exit criterion.

7. **Prove it: the double-run test.** In CI/staging: run the job twice for the same logical date, assert output equality; run day N, then N-1, then N again (out of order), assert consistency. Four lines of orchestration, the pipeline's most valuable test. Production tripwire: a duplicate-rate check on the output's natural key that pages before the CFO's dashboard does.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "We just won't re-run it" | Retry storms, infra failures, and 2am operators re-run everything eventually. 'Don't do that' is not a property of the pipeline; idempotency is. |
| "WHERE created_at > now() - 1 day is our incremental logic" | Late runs skip data, early re-runs duplicate it, backfills are meaningless. Wall-clock windows are the anti-pattern this skill exists to delete. |
| "The watermark update right after the load is close enough to atomic" | The crash between the two steps silently skips or duplicates a batch forever after. Transactional-with-the-load or the state table lies. |
| "Dedupe on all columns downstream — no producer IDs needed" | Two legitimate identical-looking events (same user, amount, second) merge into one. Producer-supplied IDs or the dedupe is lossy. |
| "The Slack notification firing twice is harmless" | Forty re-runs during an incident = forty '🎉 pipeline complete' messages = one muted channel = zero future alerting value. Side effects count (step 5). |
| "We'll write a quick backfill script for this emergency" | The fresh script diverges from the daily job's logic, and the backfilled month disagrees subtly with every other month — discovered at year-end. Same code path, parameterized. |

## Red Flags

- `now()`/`CURRENT_DATE` inside scheduled job logic.
- Append-only writes with retries enabled in the orchestrator.
- Watermark stored in a file/variable updated in a separate step from the load.
- A backfill script that isn't the daily job with different parameters.
- Overwrite-partition jobs reading "new files since last run."
- No double-run test anywhere; the invariant never stated.
- Duplicate revenue discovered by finance, not by a pipeline check.

## Verification

- [ ] Write pattern named (overwrite-partition / merge / append+dedupe-with-IDs) with its safety argument — in the pipeline README.
- [ ] Zero wall-clock time in job logic — grep for `now()|CURRENT_DATE|today()` attached, hits justified.
- [ ] Watermark transactionality shown (same transaction/atomic commit as the load) — code linked.
- [ ] Side-effect inventory with idempotence mechanism per effect — listed.
- [ ] Double-run test in CI: same-date twice + out-of-order sequence, output equality asserted — test linked, green.
- [ ] Production duplicate-rate tripwire live — check/dashboard linked.

## Example

Revenue pipeline: append-only inserts, `now()`-based window, Slack notify per run. The incident: an Airflow retry storm re-ran 6 hours of tasks → 3.2% revenue double-count discovered two weeks later in a board deck. Rebuild: daily job reads ALL source rows for the logical date (`{{ ds }}`), writes via `INSERT OVERWRITE PARTITION`; customer table switched to MERGE on `customer_id` with `updated_at` tie-break; watermark updated in the same transaction as the CDC load, 2-hour late-arrival lag; notification moved behind a per-logical-date sent-log. Double-run test added to CI (caught, immediately, that one aggregate used `CURRENT_DATE` — the test earns its keep on day one). Cleanup of the historical double-count used the new backfill path: 14 partitions rewritten, reconciliation query against billing matched to the cent.

## Related skills

- `medallion-architecture-design` — the layer map whose rebuilds depend on this property.
- `data-cleaning-pipeline` — the transformation content of these runs.
- `spark-etl-debugging` — when the re-runs themselves fail.
- `retry-and-backoff-strategy` — retries are only safe because of this skill.
