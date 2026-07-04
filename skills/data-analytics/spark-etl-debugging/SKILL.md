---
name: spark-etl-debugging
description: >
  Use when a Spark job fails or crawls — diagnosing skew, shuffle spills,
  executor OOM, stage retries from the Spark UI, and fixing the actual
  bottleneck. Triggers: "Spark job OOM", "stage stuck at 199/200",
  "shuffle spill", "executor lost", "Spark job suddenly slow", "data
  skew", "OutOfMemoryError in executor". NOT for single-node SQL
  performance (see sql-query-optimization) — this skill starts where the
  shuffle does.
---

# Spark ETL Debugging

## Overview

The Spark UI's task-duration distribution within the failing stage is the single most diagnostic view in the ecosystem — it separates skew from shuffle-bound from memory from retries in one look. Most OOMs are skew wearing a memory costume, and most skew is a NULL key wearing a data-quality costume.

## When to Use

- A Spark (or Databricks/EMR/Glue) job fails, retries, or runs far longer than its history.
- Reviewing a job before scaling its input 10×.

**When NOT to use:**
- Single-node/OLTP query tuning — `sql-query-optimization`.

## Prerequisites

- Access to the Spark UI / History Server for the affected run (the diagnosis lives there, not in the driver stacktrace).
- The job's code and its input's rough shape (sizes, key cardinalities).

## The Workflow

1. **Find the failing/slow STAGE before reading any code.** Spark UI → the long/failed stage → Tasks table sorted by duration. Read the distribution:

   ```
   Task durations within the stage:
     median 30s, max 40min, "stuck at 199/200"  → SKEW (step 3)
     uniformly slow, high shuffle read/write     → shuffle-bound (step 4)
     tasks dying (ExecutorLost / OOM)            → memory (step 5)
     whole stages rerunning (FetchFailed)        → retries/infra (step 6)
   ```

2. **Check the free fixes first:** AQE on (`spark.sql.adaptive.enabled=true` — default-on in Spark 3.2+, but jobs pinned to old configs disable it silently); statistics current; input not a swamp of tiny files (a million 100KB files = a million tasks of overhead); and **what changed** if the job was fine yesterday — input volume, key distribution (one new whale customer), cluster type, a "harmless" UDF (`production-debugging` step 1 thinking).

3. **Skew: identify the hot key, then break it.** Confirm in 30 seconds: `df.groupBy("join_key").count().orderBy(desc("count")).show(20)`. Fixes in order of cheapness:
   - AQE skew-join handling (`spark.sql.adaptive.skewJoin.enabled=true`) — often sufficient alone.
   - Hot key is NULL or a sentinel → filter/handle separately before the join (the most common skew, and it's a data bug: those rows probably shouldn't join at all — `data-cleaning-pipeline` sentinel handling).
   - One side small → broadcast join (`broadcast(dim_df)`), eliminating the shuffle (mind actual size vs `autoBroadcastJoinThreshold`).
   - Genuinely hot real keys → salting: random suffix 0–N on the hot side, explode the other side N ways, join, strip. Ugly, works, comment it.

4. **Shuffle-bound: move less data.** Shuffling 2TB to produce 2GB means the job's shape is wrong: filter and project **before** wide operations (verify pushdown actually happened in the SQL plan — UDFs and casts block it); replace `groupByKey` patterns with pre-aggregating ones; tune `spark.sql.shuffle.partitions` toward ~128–200MB per partition (AQE coalescing handles this when on); bucketing for repeated joins on the same key at scale.

5. **Memory/OOM: identify WHICH memory before adding any.** Executor OOM with huge single partitions → it's skew (back to step 3) — the most common OOM is a skew symptom, and doubling executor memory just makes the funeral more expensive. Driver OOM → a `collect()`/`toPandas()` on real data, or too many tasks' metadata — fix the code, not the driver size. Genuine executor pressure: reduce `spark.executor.cores` (fewer concurrent tasks share the same memory), check UDFs hoarding per-row objects (prefer native functions — Python UDFs cost serialization AND blind the optimizer; check `df.explain()`), uncache datasets larger than the cluster.

6. **Retries and lost executors:** FetchFailed → the *upstream* executor died, usually OOM (loop to step 5) or spot-instance reclamation (check cluster events; external shuffle service if spot is policy); deterministic failure on the same data → a poison record (binary-search the partition, add defensive parse + quarantine); speculative execution as a band-aid for straggler *nodes*, never for skew.

7. **After the fix: pin the evidence and guard the future.** Before/after stage metrics (duration, shuffle volume, spill) in the PR; a data-shape assertion where the diagnosis was data-dependent (hot-key count check before the join, input row-count bounds — `data-cleaning-pipeline` step 7's drift gates); re-run at full production scale before declaring victory — skew fixes verified on samples routinely miss the whale that only exists in prod.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "OOM — double the executor memory" | One 38GB partition doesn't care that the cluster is twice as large; the OOM is skew's symptom in most cases. The median-vs-max task check takes one minute and saves the cluster bill. |
| "The stacktrace says stage 12 failed — that's the diagnosis" | The stacktrace names the stage boundary; the UI's task table names the cause. 'Job aborted due to stage failure' is a table of contents. |
| "Sprinkle repartition(N) — it fixed a job once" | An extra full shuffle that sometimes helps by accident and always costs. Know WHY the partition count is wrong first, or you're adding load to a lottery. |
| "It's been fine for a year — must be cluster flakiness" | A year-stable job that suddenly crawls means the DATA changed (new whale key, nulled column upstream). The what-changed question (step 2) finds it; blaming infra defers it. |
| "The UDF is just a small helper function" | Python UDFs block pushdown, cost per-row serialization, and hide from the optimizer — check df.explain() for what the 'small helper' did to the plan. |
| "Skew fix verified on the 1% sample" | The whale key that causes the skew often doesn't EXIST in the sample. Full-scale verification or the fix is a hypothesis shipping to production. |

## Red Flags

- Memory raised twice for the same job with no task-distribution look.
- Diagnosis performed entirely from driver logs.
- `repartition()` calls with no comment explaining the number.
- A NULL/sentinel hot key found and salted instead of questioned (it's a data bug).
- Fix merged with no before/after stage metrics.
- The upstream data change that caused it all — never traced to its source.

## Verification

- [ ] Pathology classified from the task-duration distribution — screenshot/numbers in the PR.
- [ ] What-changed answered for sudden regressions — the input/config delta named.
- [ ] Hot-key count run for skew cases — output attached; NULL/sentinel keys dispositioned as data fixes where applicable.
- [ ] The chosen fix matches the pathology (per the step-1 table) — one line of reasoning.
- [ ] Before/after stage metrics at full production scale — both attached.
- [ ] Drift guard added where the cause was data-shape — test/assertion linked.

## Example

Nightly enrichment job: 45 min for a year, now 6h and OOMing on retries. UI: stage 12 at 199/200 for hours; task table: median 40s, max running 5h+, that task's shuffle read = 38GB vs median 90MB. Hot-key count: `join_key = NULL` — 210M rows (an upstream schema change had nulled a column feeding the key — the *actual* root cause, found by asking what changed). Fix: null keys split off pre-join (they enriched to nothing anyway — business confirmed they're unmatchable events), routed to a quarantine table; AQE skew-join enabled for residual moderate skew; drift gate added asserting null-rate on `join_key` < 1%. Runtime: 38 min. The upstream schema bug got its own ticket — and the quarantine table gave the upstream team the exact 210M-row evidence.

## Related skills

- `sql-query-optimization` — the single-node sibling; plans and pruning logic transfer.
- `data-pipeline-idempotency` — safe re-runs while you're debugging retries.
- `data-cleaning-pipeline` — the sentinel/null handling that prevents step 3's classic.
- `production-debugging` — the "what changed at T?" discipline this inherits.
