---
name: spark-etl-debugging
description: >
  Use when a Spark job fails or crawls — diagnosing skew, shuffle spills,
  executor OOM, stage retries from the Spark UI, and fixing the actual
  bottleneck. Triggers: "Spark job OOM", "stage stuck at 199/200", "shuffle
  spill", "executor lost", "Spark job suddenly slow", "data skew",
  "OutOfMemoryError in executor".
---

# Spark ETL Debugging

## When to use this skill
- A Spark (or Databricks/EMR/Glue) job fails, retries, or runs far longer than its history.
- Reviewing a job before scaling its input 10×.
- NOT for single-node SQL performance — that's `sql-query-optimization`; this skill starts where the shuffle does.

## Prerequisites
- Access to the Spark UI / History Server for the affected run (the diagnosis lives there, not in the driver stacktrace).
- The job's code and its input's rough shape (sizes, key cardinalities).

## Workflow

1. **Find the failing/slow STAGE before reading any code.** Spark UI → Jobs → the long/failed stage → Tasks table sorted by duration. The single most diagnostic view in Spark: **task duration distribution within the stage.** Read it as:
   - Median task 30s, max task 40min, stage "stuck at 199/200" → **skew** (step 3).
   - All tasks uniformly slow with high shuffle read/write → shuffle-bound (step 4).
   - Tasks dying with ExecutorLostFailure/OOM → memory (step 5).
   - Whole stages rerunning → retries/fetch failures (step 6).

2. **Check the free fixes first:** AQE on (`spark.sql.adaptive.enabled=true` — handles moderate skew and shuffle-partition sizing automatically; default-on in Spark 3.2+, but jobs pinned to old configs disable it silently); statistics current for the optimizer; input not a swamp of tiny files (a million 100KB files = a million tasks of overhead — compact upstream or `maxPartitionBytes` tuning); and **what changed** if the job was fine yesterday — input volume, key distribution (one new whale customer), cluster instance type, a "harmless" code change adding a UDF (`production-debugging` step 1 thinking).

3. **Skew: identify the hot key, then break it.** Confirm with a 30-second count: `df.groupBy("join_key").count().orderBy(desc("count")).show(20)` — one key with 100M rows while the median has 1k is your 40-minute task. Fixes in order of cheapness:
   - AQE skew-join handling (`spark.sql.adaptive.skewJoin.enabled=true`) — often sufficient alone.
   - Hot key is NULL or a sentinel (`"unknown"`) → filter/handle separately before the join (the most common skew, and it's a data bug: half those rows probably shouldn't join at all — `data-cleaning-pipeline` sentinel handling).
   - One side small → broadcast join (`broadcast(dim_df)`), eliminating the shuffle entirely (mind `autoBroadcastJoinThreshold` and actual size).
   - Genuinely hot real keys → salting: append a random suffix 0–N to the hot side's key, explode the other side N ways, join, strip. Ugly, works, comment it (`# salted join: customer 8812 is 40% of events`).

4. **Shuffle-bound: move less data.** Check the stage's shuffle read/write sizes — shuffling 2TB to produce 2GB means the job's shape is wrong: filter and project **before** the wide operations (predicate/column pruning — verify in the SQL plan that pushdown actually happened; UDFs and casts block it); replace `groupByKey`-style patterns with pre-aggregating ones (`reduceByKey`/SQL aggregations); tune `spark.sql.shuffle.partitions` to target ~128–200MB per partition (the 200 default is wrong at both ends of scale — AQE coalescing handles this when on); for repeated joins on the same key at scale, consider bucketing the tables.

5. **Memory/OOM: identify WHICH memory before adding any.** Executor OOM with huge single partitions → it's skew (back to step 3) — the most common OOM is a skew symptom, and doubling executor memory just makes the funeral more expensive. Driver OOM → a `collect()`/`toPandas()` on real data or too many small tasks' metadata — fix the code, not the driver size. Genuine executor pressure: reduce `spark.executor.cores` (fewer concurrent tasks share the same memory), check UDFs hoarding per-row objects (and prefer native functions/pandas UDFs over Python UDFs — serialization overhead AND optimizer blindness), watch for `cache()` of datasets larger than the cluster (uncache or checkpoint what's actually reused).

6. **Retries and lost executors:** FetchFailed → the *upstream* executor died, usually OOM (loop to step 5) or spot-instance reclamation (check cluster events; decouple shuffle survival with external/persistent shuffle service if spot is policy); tasks that fail deterministically on the same data → a poison record (malformed row crashing a UDF — binary-search the partition, add the defensive parse + quarantine); speculative execution (`spark.speculation`) as a band-aid for straggler *nodes*, never for skew.

7. **After the fix: pin the evidence and guard the future.** Record before/after stage metrics (duration, shuffle volume, spill) in the PR; add a data-shape assertion where the diagnosis was data-dependent (hot-key count check that warns before the join, input row-count bounds — `data-cleaning-pipeline` step 7's drift gates); and re-run at full production scale before declaring victory — skew fixes verified on samples routinely miss the whale that only exists in prod.

## Common pitfalls
- Reading the driver's stacktrace instead of the Spark UI. The stacktrace names the stage boundary; the UI names the cause. `Job aborted due to stage failure` is a table of contents, not a diagnosis.
- Throwing memory/instances at skew. One 40GB partition doesn't care that your cluster is now twice as large — the max-task-vs-median-task check takes one minute and saves the cluster bill.
- `repartition(N)` sprinkled as a magic incantation — an extra full shuffle that sometimes helps by accident and always costs. Know *why* the partition count is wrong first.
- Doubling `shuffle.partitions` when the problem was a NULL hot key. More partitions, same 100M-row key in one of them.
- UDF-first habits: Python UDFs block predicate pushdown, cost serialization per row, and hide from the optimizer — check `df.explain()` for what your "small helper function" did to the plan.
- Fixing without the what-changed question: the job was fine for a year; the fix belongs to the *input's* new shape (new whale key), and tomorrow's other job hits the same whale. Trace the data change to its source.

## Example
Nightly enrichment job: 45 min for a year, now 6h and OOMing on retries. UI: stage 12 at 199/200 for hours; task table: median 40s, max running 5h+, that task's shuffle read = 38GB vs median 90MB. Hot-key count: `join_key = NULL` — 210M rows (an upstream schema change had nulled a column feeding the key — the *actual* root cause, found by asking what changed). Fix: null keys split off pre-join (they enriched to nothing anyway — business confirmed they're unmatchable events), routed to a quarantine table; AQE skew-join enabled for residual moderate skew; drift gate added asserting null-rate on `join_key` < 1%. Runtime: 38 min. The upstream schema bug got its own ticket — and the quarantine table gave the upstream team the exact 210M-row evidence.

## Related skills
- `sql-query-optimization` — the single-node sibling; plans and pruning logic transfer.
- `data-pipeline-idempotency` — safe re-runs while you're debugging retries.
- `data-cleaning-pipeline` — the sentinel/null handling that prevents step 3's classic.
- `production-debugging` — the "what changed at T?" discipline this inherits.
