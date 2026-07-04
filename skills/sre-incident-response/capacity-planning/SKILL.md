---
name: capacity-planning
description: >
  Use when forecasting whether a system can handle expected load —
  headroom targets, load-test interpretation, finding the true
  bottleneck, planning for events. Triggers: "will we survive Black
  Friday", "capacity plan", "how much headroom", "load test results",
  "when do we need to scale", "traffic is doubling". NOT for real-time
  scaling during an incident (see incident-triage) — this is the work
  that prevents it.
---

# Capacity Planning

## Overview

Capacity is a chain, and the system's limit is its weakest link's — which is frequently not the component being watched. Forecast demand in business terms, find each service's real constraint, load-test to the knee not to a pass, and write the plan as triggers rather than dates.

## When to Use

- A traffic event approaches (launch, sale, marketing push) or growth trends toward a wall.
- Interpreting load-test results into a scaling decision.

**When NOT to use:**
- Mid-incident emergency scaling — `incident-triage` mitigation.

## Prerequisites

- Utilization and traffic telemetry with at least weeks of history.
- The SLOs the capacity must defend (`slo-definition`) — capacity is meaningless without a "good enough" line.

## The Workflow

1. **Forecast demand in business terms first, then translate.** Growth rate from trend (fit on peak, not average traffic), plus event multipliers from the business (marketing's expected uplift, last year's event ratio). Translate to system units via measured ratios: users → requests/s → queries/s per service. Document the assumptions — the forecast will be wrong; knowing *which assumption* broke is the value.

2. **Find the real constraint per service — capacity is a chain, not a number.** For each system in the critical path, identify the first thing that breaks as load rises: CPU, memory, connection pools, DB IOPS, a lock, a third-party rate limit, a hard quota (cloud limits count!). Past incidents and saturation metrics (queue depths, pool wait times) point at it.

3. **Load test to find the knee, not to pass.** Ramp load steadily against a prod-parity environment (`environment-parity`) while watching latency percentiles: capacity is where p99 leaves the SLO, *not* where errors start — systems degrade before they fail. Test with production-shaped traffic (real endpoint mix, real payload sizes, realistic cache hit rates — a cache-warm synthetic test overstates capacity dramatically), and with retry-realistic clients: at saturation, timeouts trigger client retries, so 1.1× organic load becomes 2× offered load.

4. **Set headroom by how fast you can add capacity.** Rule: peak forecast load ≤ (capacity × utilization target), where the target reflects reaction time — 50–60% for stateful/slow-to-scale systems (DBs: scaling is a project), 70–75% for autoscaled stateless fleets (scaling is minutes), and always enough surviving-N-minus-1: losing one AZ/node at peak must keep you under the knee.

5. **Check the non-obvious dimensions:** data growth (that query is fine at 10M rows, the plan flips at 100M), connection counts (each new replica multiplies DB connections — poolers before replicas), queue/backlog drain time after a blip at peak (a 10-min outage at 2× traffic needs the consumers to catch up — can they?), and third-party limits (payment provider TPS, email quotas — contractual ceilings are capacity too).

6. **Write the plan as triggers, not dates:** "add a read replica when sustained peak QPS > 8k (currently 5.2k, trend says ~October)" — with lead time built in (procurement, migration, warm-up all take longer than the graph gives you). Cheap insurance for events: pre-scale + pre-warm before, load-shedding and graceful-degradation switches ready in case the forecast lowballs.

7. **Close the loop after every peak:** forecast vs actual demand, predicted vs actual bottleneck, headroom consumed. Each miss tunes next cycle's model — capacity planning is a quarterly loop, not an annual document.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "We're at 40% average utilization — plenty of room" | Capacity problems live at peak-of-peak: 40% average coexists with 95% every day at 9am. Plan on the peak percentile or plan on fiction. |
| "The app tier autoscales, so we can handle anything" | More app servers deliver the overload to the database faster. The chain's weakest link sets capacity, and it's usually the stateful thing that doesn't autoscale. |
| "Load test passed at 2× — we're good for 2×" | Passed-with-mocked-DB, cache-warm, and no client retries overstates real capacity several-fold. And 'passed' isn't the question — where's the KNEE? |
| "Linear extrapolation: 2× traffic needs 2× resources" | Lock contention, GC pressure, and O(n²) paths make scaling nonlinear past the knee — which is why finding the knee (step 3) precedes any arithmetic. |
| "The cloud is elastic — we'll scale when we see load coming" | Instance-type availability, account quotas, and IP space have all ended real Black Fridays. Elastic has ceilings; check quotas in the plan. |
| "The vendor handles our payment traffic — not our capacity problem" | Their contractual TPS is your hard limit, and it's usually sized to yesterday's volume. Third-party ceilings are capacity findings (the example's first one). |

## Red Flags

- Capacity discussion citing average utilization only.
- Load tests that mock the database or run cache-warm.
- No one can name the next bottleneck for the critical path's top service.
- Scaling plans expressed as dates with no metric triggers.
- Cloud quotas never checked against the event forecast.
- Post-event: no forecast-vs-actual review anywhere.

## Verification

- [ ] Demand forecast with named assumptions and business inputs — doc linked.
- [ ] Constraint identified per critical-path service — the chain table exists.
- [ ] Knee measured at prod parity with production-shaped, retry-realistic traffic — load-test report linked.
- [ ] Headroom target per service justified by reaction time; N-1 check passes — numbers shown.
- [ ] Triggers written with current values and lead times — plan linked.
- [ ] Post-peak review completed last cycle (forecast vs actual) — findings recorded.

## Example

E-commerce, Black Friday forecast 4× normal peak. Step 2 chain review: app tier autoscales fine; constraint candidates were Postgres (write IOPS) and the payment provider (contractual 500 TPS ≈ 3.2× peak — first finding, renegotiated to 800). Load test at prod parity: p99 left SLO at 2.8× current peak; bottleneck was *connection pool wait*, not CPU — pgbouncer deployed, retest knee moved to 5.1×. Plan: pre-scale to hold 4× at 65% utilization, load-shedding flag on the recommendation widget (sheds 30% DB load in one flip), N-1 AZ check passed. Actual event: 4.6× peak (forecast missed by 15% — marketing's email did better than claimed), shedding flag used for 40 minutes, SLO held. Post-peak review updated the email-uplift assumption.

## Related skills

- `slo-definition` — the line capacity defends.
- `environment-parity` — why the load-test environment can be trusted.
- `production-debugging` — reading the saturation signals in step 2.
- `retry-and-backoff-strategy` — the retry-storm multiplier's mechanics.
