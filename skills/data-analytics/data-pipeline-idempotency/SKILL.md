---
name: data-pipeline-idempotency
description: >
  Use when making a data pipeline safe to re-run — upserts, overwrite-partition
  semantics, watermarks, backfills that don't double-count. Triggers:
  "pipeline reran and duplicated data", "make this job idempotent",
  "safe to backfill?", "watermark for incremental load", "rerun the DAG",
  "exactly-once processing".
---

# Data Pipeline Idempotency

## When to use this skill
- Building or fixing any scheduled/triggered pipeline — because every pipeline WILL be re-run (retries, backfills, operator panic).
- A re-run just double-counted and you're cleaning up.
- NOT for one-off exploratory transforms — though the moment a "one-off" gets a schedule, this applies.

## Prerequisites
- The pipeline's write pattern (append? merge? overwrite?) and its natural key or partition scheme.

## Workflow

1. **State the invariant: same logical input → same output state, no matter how many runs.** Test mentally (then actually): run the job twice for the same day — is the result identical to running once? If the answer involves "well, don't do that," the pipeline is a scheduled incident.

2. **Pick the write pattern by what the job produces — the two safe shapes:**
   - **Overwrite-by-partition** (the workhorse for batch): the job owns a partition (usually a date) and replaces it wholesale — `INSERT OVERWRITE PARTITION (ds='2026-07-02')`, Delta `replaceWhere`, BigQuery partition decorator writes. Re-run = same partition rewritten = idempotent by construction. Requires: the partition key is derivable from the run's logical date, and the job reads *all* the input for that partition (not "whatever's new").
   - **Merge/upsert on a natural key** (for entity tables and CDC): `MERGE INTO ... ON target.id = source.id WHEN MATCHED UPDATE WHEN NOT MATCHED INSERT`, with a deterministic tie-break (latest `updated_at` wins) so replayed batches converge.
   Blind `INSERT`/append is the unsafe shape — every retry is a duplicate. If append is forced (immutable event stores), dedupe downstream on an event ID (which the producer must supply — idempotency starts at the source).

3. **Separate logical time from wall-clock time.** The job for `2026-07-02` processes data *belonging to* that date, regardless of when it runs — the run date comes from the scheduler's logical/execution date parameter, never `datetime.now()`. `now()`-based jobs produce different results on re-run by definition, and backfills become impossible. Same rule for `CURRENT_DATE` inside the SQL.

4. **Incremental loads: watermark with a lag, tolerate overlap.** Track high-water mark (max `updated_at` processed, or CDC offset) in a state table updated *transactionally with the load itself* (state advanced but load failed = skipped data; load done but state not advanced = reprocessing — the second is why step 2's merge matters: overlap must be harmless). Subtract a late-arrival lag from the watermark (`WHERE updated_at > :wm - INTERVAL '1 hour'`) so slightly-late records get caught by the overlap-tolerant merge instead of lost forever.

5. **Make side effects idempotent too — the forgotten half.** The data write is idempotent but the job also sends emails, fires webhooks, or bumps an external counter → the re-run spams customers. Options: idempotency keys on the external calls (`api-design-rest` step 5 from the consumer side), a sent-log checked before sending, or move side effects to a downstream consumer of the *data* (which sees one state, not N runs). A pipeline is as idempotent as its worst side effect.

6. **Design backfills as first-class:** parameterized by date range, running the same code path as the daily run (a separate "backfill script" diverges within months and becomes the thing that double-counts), bounded parallelism (30 partition-overwrites at once is a self-DDoS on the warehouse — and beware cross-partition dependencies like cumulative metrics, which force sequential order), and a post-backfill reconciliation query (row counts and totals per day vs source) as the exit criterion.

7. **Prove it in CI/staging: the double-run test.** Run the job twice for the same logical date, assert byte/row equality of output; run day N, then N-1, then N again (out of order), assert consistency. This is the pipeline's most valuable test and it's four lines of orchestration. Add the production tripwire: a duplicate-rate check on the output's natural key (`dbt` unique test / a daily count) that pages before the CFO's dashboard does.

## Common pitfalls
- `WHERE created_at > now() - interval '1 day'` as "incremental logic": late runs skip data, early re-runs duplicate it, and backfilling is meaningless. Logical dates + watermarks exist for this.
- Watermark stored in a file/variable updated *after* commit in a separate step — the crash between the two steps silently skips or duplicates a batch forever after.
- Partition-overwrite jobs that read incrementally ("new files since last run") but overwrite the whole partition — the re-run overwrites yesterday's full partition with only today's slice of it. Overwrite semantics require full-partition reads.
- The dedupe-downstream job that dedupes on all columns instead of the event key — two legitimate identical-looking events (same user, same amount, same second) merged into one. Producer-supplied IDs or bust.
- Idempotent SQL, non-idempotent Slack notification — 40 re-runs during an incident, 40 "pipeline complete 🎉" messages, one muted channel, zero future alerting value.
- Backfill script written fresh during the emergency, diverging from the daily job's logic — the backfilled month disagrees subtly with every other month, discovered at year-end.

## Example
Revenue pipeline: append-only inserts, `now()`-based window, Slack notify per run. The incident: an Airflow retry storm re-ran 6 hours of tasks → 3.2% revenue double-count discovered two weeks later in a board deck. Rebuild: daily job reads ALL source rows for the logical date (`{{ ds }}`), writes via `INSERT OVERWRITE PARTITION`; entity-side customer table switched to MERGE on `customer_id` with `updated_at` tie-break; watermark table updated in the same transaction as the CDC load, 2-hour late-arrival lag; notification moved behind a per-logical-date sent-log. Double-run test added to CI (caught, immediately, that one aggregate used `CURRENT_DATE` — the test earns its keep on day one). Cleanup of the historical double-count used the new backfill path: 14 partitions rewritten, reconciliation query against billing matched to the cent.

## Related skills
- `medallion-architecture-design` — the layer map whose rebuilds depend on this property.
- `data-cleaning-pipeline` — the transformation content of these runs.
- `spark-etl-debugging` — when the re-runs themselves fail.
- `error-handling-strategy` — retries are only safe because of this skill.
