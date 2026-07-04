---
name: sql-query-optimization
description: >
  Use when a SQL query is slow — reading execution plans, index strategy,
  rewriting antipatterns, knowing when the problem isn't the query.
  Triggers: "this query is slow", "EXPLAIN output", "needs an index?",
  "query timeout", "optimize this SQL", "sequential scan on a huge
  table". NOT for distributed engines' skew/shuffle problems (see
  spark-etl-debugging) — this skill is OLTP-shaped (Postgres/MySQL) first.
---

# SQL Query Optimization

## Overview

The plan with actual numbers — not the query text, not intuition — says where the time goes; everything else follows from reading it at production scale. Half of "slow query" work is stale statistics, non-sargable predicates, and ORM surprises, none of which need a new index.

## When to Use

- A specific query (or ORM-generated pattern) is measurably slow.
- Reviewing schema/index changes meant to fix performance.

**When NOT to use:**
- Distributed engines (Spark/warehouse skew, shuffles) — `spark-etl-debugging`.

## Prerequisites

- Access to run `EXPLAIN ANALYZE` against realistic data volume — plans on a 1k-row dev DB are fiction (`environment-parity` step 4).

## The Workflow

1. **Get the real plan with real numbers:**
   ```sql
   EXPLAIN (ANALYZE, BUFFERS) SELECT ...;
   ```
   Read for the three classic signals: (a) **actual rows vs estimated rows off by 100×+** — stale statistics (`ANALYZE tablename;` is free and fixes more than people expect); (b) the node where **actual time concentrates** — optimize that node, not the query you *think* is slow; (c) **Seq Scan on big tables under a selective WHERE** — the index conversation starts here, not before.

2. **Check whether the query asks for too much:** `SELECT *` dragging 40 columns through a sort (and off the table for index-only scans), missing `LIMIT` on "show recent" queries, `OFFSET 100000` pagination (scans and discards — switch to keyset: `WHERE (created_at, id) < (?, ?) ORDER BY ... LIMIT 50`), and N+1 patterns from the ORM (one query per row in app logs = the fix is a join/`IN`, not an index). Log the ORM's actual SQL — the "one simple query" is often 1+N or a monster join you never wrote.

3. **Index to match the access pattern, composite in the right order:** equality columns first, then the range/sort column: `(tenant_id, status, created_at)` serves `WHERE tenant_id=? AND status=? ORDER BY created_at DESC`. Rules that pay: the leading column must appear in the WHERE; an index on `(a,b)` makes a separate `(a)` mostly redundant; covering (`INCLUDE`) columns buy index-only scans; partial indexes (`WHERE status='active'`) for skewed flags. Verify adoption with a re-EXPLAIN — creating an index the planner ignores is the most common "optimization."

4. **Hunt the non-sargable predicates — reasons the planner CAN'T use your index:** functions wrapping the column (`WHERE DATE(created_at) = '2026-07-01'` → rewrite as a range), type mismatches (string column vs int — silent cast kills the index; also an app bug), leading-wildcard `LIKE '%term'` (needs trigram/FTS), `OR` across different columns (often better as `UNION ALL`), and `NOT IN` with nullable subqueries (also a correctness trap — `NOT EXISTS`).

5. **Restructure when the shape is the problem:** correlated subqueries executing per-row → joins or window functions (`ROW_NUMBER() OVER (PARTITION BY ...)` replaces the "latest row per group" self-join); `DISTINCT` slapped on to hide a fan-out join (fix the join — the grain, `exploratory-data-analysis` step 1); CTE materialization walls in older Postgres blocking predicate pushdown.

6. **When the plan is fine and it's still slow, look around the query:** lock waits (`pg_locks` — different fix entirely), connection pool exhaustion (queue time billed as query time), cold cache vs warm (BUFFERS read vs hit), bloat from update-heavy tables (autovacuum), or "the query runs 400×/minute and should be cached." The EXPLAIN told the query's truth; the system has other truths.

7. **Fix, measure, and leave guardrails:** before/after `EXPLAIN ANALYZE` at production scale; `pg_stat_statements` to confirm the aggregate win (total time, not the one execution); slow-query log threshold as the regression tripwire; the index documented next to the query it serves so the next migration doesn't drop it as "unused."

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "Just add an index on the WHERE column" | The predicate may be non-sargable (the index can't serve `DATE(col)=`), the planner may ignore it, and every index taxes every write. The plan tells you IF an index is even the problem. |
| "It's fast on my dev database" | 1k rows seq-scan CORRECTLY — faster than the index there. Dev-scale plans predict nothing; realistic volume or the whole exercise is theater. |
| "EXPLAIN looks fine — estimated costs are low" | Estimates are exactly what's wrong when statistics are stale (the example was off 400×). ANALYZE-with-actuals or you're reading the planner's fiction. |
| "DISTINCT fixed the duplicate rows" | It hid a fan-out join — slower AND wrong (the grain bug survives, disguised). Fix the join; DISTINCT-as-bandage is the tell. |
| "The ORM generates fine SQL, no need to look" | The innocent `.orders` attribute is an N+1 generating 400 queries per page. Log the real SQL; ORMs are trusted the way teenagers are trusted with cars. |
| "Force the plan with hints — the planner is dumb" | Hints rot as data changes and pin today's workaround into tomorrow's regression. Fix statistics/indexes/shape so the planner WANTS the good plan; hints are the last resort, dated and commented. |

## Red Flags

- Index migrations shipped with no before/after plan attached.
- `EXPLAIN` (estimates-only) cited as evidence.
- Query "optimized" on dev-scale data.
- `DISTINCT` or `GROUP BY` added in the same commit that "fixed" duplicates.
- App logs showing per-row query storms nobody has counted.
- Indexes accumulating with no `pg_stat_user_indexes` usage review.

## Verification

- [ ] `EXPLAIN (ANALYZE, BUFFERS)` before AND after, at production-representative volume — both attached.
- [ ] The bottleneck node identified by actual time — named in the PR.
- [ ] Statistics freshness checked (`ANALYZE` run) before structural changes.
- [ ] New indexes verified adopted by the plan — re-EXPLAIN shown; write-cost acknowledged.
- [ ] `pg_stat_statements` aggregate confirms the win (total time delta) — numbers quoted.
- [ ] Guardrails placed: slow-query threshold + index documented at its query.

## Example

Dashboard query timing out at 30s: latest status per device for one customer, `SELECT * FROM readings WHERE customer_id=? AND DATE(ts)=CURRENT_DATE` + app-side dedupe. EXPLAIN ANALYZE: seq scan (140M rows), estimate off 400× (stale stats), the `DATE()` wrapper blocking the existing `(customer_id, ts)` index. Fixes in order: `ANALYZE readings`; rewrite predicate to `ts >= CURRENT_DATE AND ts < CURRENT_DATE + 1` (index now eligible); replace app-side dedupe with `ROW_NUMBER() OVER (PARTITION BY device_id ORDER BY ts DESC)` filtered to 1; `SELECT` only the 6 used columns. Result: 30s → 90ms, no new index needed. pg_stat_statements a week later: total time for the pattern down 99.3%; one guardrail added — slow-query log at 1s caught the *next* dashboard's copy-pasted `DATE()` a month later.

## Related skills

- `spark-etl-debugging` — the distributed-engine sibling of this skill.
- `capacity-planning` — when the query is fine and the system is the constraint.
- `exploratory-data-analysis` — grain problems masquerading as performance problems.
