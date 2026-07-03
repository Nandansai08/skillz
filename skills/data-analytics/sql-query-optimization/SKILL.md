---
name: sql-query-optimization
description: >
  Use when a SQL query is slow — reading execution plans, index strategy,
  rewriting antipatterns, knowing when the problem isn't the query.
  Triggers: "this query is slow", "EXPLAIN output", "needs an index?",
  "query timeout", "optimize this SQL", "sequential scan on a huge table".
---

# SQL Query Optimization

## When to use this skill
- A specific query (or query pattern from an ORM) is measurably slow.
- Reviewing schema/index changes meant to fix performance.
- NOT for warehouse/columnar engines' distributed problems (partitioning/clustering apply, but skew and shuffle live in `spark-etl-debugging` territory) — this skill is OLTP-shaped (Postgres/MySQL) first.

## Prerequisites
- Access to run `EXPLAIN ANALYZE` (or the engine's equivalent) against realistic data volume — plans on a 1k-row dev DB are fiction (`environment-parity` step 4).

## Workflow

1. **Get the real plan with real numbers:**
   ```sql
   EXPLAIN (ANALYZE, BUFFERS) SELECT ...;
   ```
   `ANALYZE` executes and shows actual times/rows; `BUFFERS` shows I/O. Read for the three classic signals: (a) **actual rows vs estimated rows off by 100×+** — stale statistics or correlated columns misleading the planner (`ANALYZE tablename;` first, it's free and fixes more than people expect); (b) the node where **actual time concentrates** — optimize that node, not the query you *think* is slow; (c) **Seq Scan on big tables under a selective WHERE** — the index conversation starts here, not before.

2. **Check whether the query asks for too much:** `SELECT *` dragging 40 columns through a sort (and off the table for index-only scans), missing `LIMIT` on "show recent" queries, `OFFSET 100000` pagination (scans and discards — switch to keyset: `WHERE (created_at, id) < (?, ?) ORDER BY ... LIMIT 50`), and N+1 patterns from the ORM (one query per row in app logs = the fix is a join/`IN`, not an index — `code-review-checklist` step 4).

3. **Index to match the access pattern, composite in the right order:** equality columns first, then the range/sort column: `(tenant_id, status, created_at)` serves `WHERE tenant_id=? AND status=? ORDER BY created_at DESC`. Rules that pay: the leading column must appear in the WHERE for the index to serve; an index on `(a,b)` makes a separate `(a)` mostly redundant; covering (`INCLUDE`) columns buy index-only scans for hot queries; partial indexes (`WHERE status='active'`) for skewed flags. Verify adoption with a re-EXPLAIN — creating an index the planner ignores is the most common "optimization."

4. **Hunt the non-sargable predicates — reasons the planner CAN'T use your index:** functions wrapping the column (`WHERE DATE(created_at) = '2026-07-01'` → rewrite as a range `>= ... AND < ...`), type mismatches (string column compared to int — silent cast kills the index; also an app bug), leading-wildcard `LIKE '%term'` (needs trigram/FTS indexes instead), `OR` across different columns (often better as `UNION ALL` of two indexed queries), and `NOT IN` with nullable subqueries (also a correctness trap — `NOT EXISTS`).

5. **Restructure when the shape is the problem:** correlated subqueries executing per-row → joins or window functions (`ROW_NUMBER() OVER (PARTITION BY ...)` replaces the classic "latest row per group" self-join); `DISTINCT` slapped on to hide a fan-out join (fix the join — see the grain, `exploratory-data-analysis` step 1); CTE materialization walls in older Postgres (≤11, or `MATERIALIZED` keyword) blocking predicate pushdown.

6. **When the plan is fine and it's still slow, look around the query:** lock waits (`pg_locks`, blocked on another transaction — different fix entirely), connection pool exhaustion (queue time billed as query time — `capacity-planning`'s pgbouncer finding), cold cache vs warm (BUFFERS read vs hit), bloat from update-heavy tables (autovacuum tuning), or simply "the query runs 400×/minute and should be cached." The EXPLAIN told you the query's truth; the system has other truths.

7. **Fix, measure, and leave guardrails:** before/after with `EXPLAIN ANALYZE` at production scale AND under production concurrency where possible; `pg_stat_statements` to confirm the aggregate win (total time, not just the one execution); and the regression guard — a slow-query log threshold plus the index documented next to the query it serves, so the next migration doesn't drop it as "unused."

## Common pitfalls
- Indexing every column that appears in any WHERE. Each index taxes every write and bloats memory; indexes are bought with write throughput. Design for the top query patterns, delete unused ones (`pg_stat_user_indexes`).
- Optimizing on the dev database: 1k rows seq-scans *correctly* (it's faster there), so the plan tells you nothing about prod. Realistic volume or nothing.
- Reading estimated plans (`EXPLAIN` alone) as truth — the interesting failures are exactly where estimates and actuals diverge.
- Adding `DISTINCT`/`GROUP BY` to silence duplicate rows from a bad join. Slower AND wrong — the fan-out is the bug.
- Believing the ORM is innocent. Log the actual SQL (`echo=True`, `django.db.backends` logging); the "one simple query" is often 1+N or a monster join you never wrote.
- Hint-forcing the planner as a first resort — hints rot as data changes; fix statistics/indexes/shape so the planner *wants* the good plan.

## Example
Dashboard query timing out at 30s: latest status per device for one customer, `SELECT * FROM readings WHERE customer_id=? AND DATE(ts)=CURRENT_DATE` + app-side dedupe. EXPLAIN ANALYZE: seq scan (140M rows), estimate off 400× (stale stats), the `DATE()` wrapper blocking the existing `(customer_id, ts)` index. Fixes in order: `ANALYZE readings`; rewrite predicate to `ts >= CURRENT_DATE AND ts < CURRENT_DATE + 1` (index now eligible); replace app-side dedupe with `ROW_NUMBER() OVER (PARTITION BY device_id ORDER BY ts DESC)` filtered to 1; `SELECT` only the 6 used columns. Result: 30s → 90ms, no new index needed. pg_stat_statements a week later: total time for the pattern down 99.3%; one guardrail added — slow-query log at 1s caught the *next* dashboard's copy-pasted `DATE()` a month later.

## Related skills
- `spark-etl-debugging` — the distributed-engine sibling of this skill.
- `capacity-planning` — when the query is fine and the system is the constraint.
- `exploratory-data-analysis` — grain problems masquerading as performance problems.
