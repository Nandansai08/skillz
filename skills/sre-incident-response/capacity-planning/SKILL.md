---
name: capacity-planning
description: >
  Use when forecasting whether a system can handle expected load — headroom
  targets, load-test interpretation, finding the true bottleneck, planning
  for events. Triggers: "will we survive Black Friday", "capacity plan",
  "how much headroom", "load test results", "when do we need to scale",
  "traffic is doubling".
---

# Capacity Planning

## When to use this skill
- A traffic event approaches (launch, sale, marketing push) or growth trends toward a wall.
- Interpreting load-test results into a scaling decision.
- NOT for real-time scaling during an incident — that's `incident-triage` mitigation; this is the work that prevents it.

## Prerequisites
- Utilization and traffic telemetry with at least weeks of history.
- The SLOs the capacity must defend (`slo-definition`) — capacity is meaningless without a "good enough" line.

## Workflow

1. **Forecast demand in business terms first, then translate.** Growth rate from trend (fit on peak, not average traffic), plus event multipliers from the business (marketing's expected uplift, last year's event ratio). Translate to system units via measured ratios: users → requests/s → queries/s per service. Document the assumptions — the forecast will be wrong; knowing *which assumption* broke is the value.

2. **Find the real constraint per service — capacity is a chain, not a number.** For each system in the critical path, identify the first thing that breaks as load rises: CPU, memory, connection pools, DB IOPS, a lock, a third-party rate limit, a hard quota (cloud limits count!). The system's capacity is its weakest link's, and it's frequently not the one being watched. Past incidents and saturation metrics (queue depths, pool wait times) point at it.

3. **Load test to find the knee, not to pass.** Ramp load steadily against a prod-parity environment (`environment-parity`) while watching latency percentiles: capacity is where p99 leaves the SLO, *not* where errors start — systems degrade before they fail. Test with production-shaped traffic (real endpoint mix, real payload sizes, realistic cache hit rates — a cache-warm synthetic test overstates capacity dramatically). Record the knee per constraint from step 2.

4. **Set headroom by how fast you can add capacity.** Rule: peak forecast load ≤ (capacity × utilization target), where the target reflects reaction time — 50–60% for stateful/slow-to-scale systems (DBs: scaling is a project), 70–75% for autoscaled stateless fleets (scaling is minutes), and always enough surviving-N-minus-1: losing one AZ/node at peak must keep you under the knee.

5. **Check the non-obvious dimensions:** data growth (that query is fine at 10M rows, the plan flips at 100M), connection counts (each new replica multiplies DB connections — poolers before replicas), queue/backlog drain time after a blip at peak (a 10-min outage at 2× traffic needs the consumers to catch up — can they?), and third-party limits (payment provider TPS, email quotas).

6. **Write the plan as triggers, not dates:** "add a read replica when sustained peak QPS > 8k (currently 5.2k, trend says ~October)" — with lead time built in (procurement, migration, warm-up all take longer than the graph gives you). Cheap insurance for events: pre-scale + pre-warm before, load-shedding and graceful-degradation switches ready (`deployment-rollback-plan` step 4 style flags) in case the forecast lowballs.

7. **Close the loop after every peak:** forecast vs actual demand, predicted vs actual bottleneck, headroom consumed. Each miss tunes next cycle's model — capacity planning is a quarterly loop, not an annual document.

## Common pitfalls
- Planning on average load. Capacity problems live at peak-of-peak (the top 5 minutes of the year); a system at 40% average can be at 95% every day at 9am.
- Load-testing the app tier while mocking the DB — testing the part that scales easily and skipping the constraint. Test through the chain.
- Linear extrapolation past a knee: 2× traffic ≠ 2× resources when lock contention, GC, or O(n²) paths kick in. The knee from step 3 is where linearity dies.
- Forgetting the retry storm multiplier: at saturation, timeouts trigger client retries, so 1.1× organic load becomes 2× offered load. Load-shed *before* the knee, and test with retry-realistic clients.
- Cloud ≠ infinite: instance-type availability, account quotas, and IP space have all ended Black Fridays. Check quotas in the plan.
- Scaling stateless tiers while the DB was the constraint — more app servers just deliver the overload faster.

## Example
E-commerce, Black Friday forecast 4× normal peak. Step 2 chain review: app tier autoscales fine; constraint candidates were Postgres (write IOPS) and the payment provider (contractual 500 TPS ≈ 3.2× peak — first finding, renegotiated to 800). Load test at prod parity: p99 left SLO at 2.8× current peak; bottleneck was *connection pool wait*, not CPU — pgbouncer deployed, retest knee moved to 5.1×. Plan: pre-scale to hold 4× at 65% utilization, load-shedding flag on the recommendation widget (sheds 30% DB load in one flip), N-1 AZ check passed. Actual event: 4.6× peak (forecast missed by 15% — marketing's email did better than claimed), shedding flag used for 40 minutes, SLO held. Post-peak review updated the email-uplift assumption.

## Related skills
- `slo-definition` — the line capacity defends.
- `environment-parity` — why the load-test environment can be trusted.
- `production-debugging` — reading the saturation signals in step 2.
