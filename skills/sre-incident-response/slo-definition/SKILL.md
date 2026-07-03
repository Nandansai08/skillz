---
name: slo-definition
description: >
  Use when defining service reliability targets — choosing SLIs, setting SLO
  numbers, running error budgets, deciding what reliability to promise.
  Triggers: "define SLOs", "what should our uptime target be", "error
  budget", "which SLIs", "how many nines", "reliability targets for this service".
---

# SLO Definition

## When to use this skill
- Establishing reliability targets for a service (new or existing).
- An error budget exists on paper but changes nothing about decisions.
- NOT for building the alerts on top — that's `alerting-design`; this produces what those alerts defend.

## Prerequisites
- Telemetry good enough to *measure* candidate SLIs for a few weeks (you set targets from data, not aspiration).
- The service's critical user journeys, named.

## Workflow

1. **Pick SLIs per user journey, not per component.** For each critical journey ("user searches and gets results"), choose 1–2 indicators from the standard menu: **availability** (good requests / total requests), **latency** (proportion of requests faster than a threshold — not the percentile value itself; "99% of requests < 400ms" is measurable as a ratio), **freshness/correctness** for pipelines (see `data-pipeline-idempotency` world). Measure as close to the user as possible: load-balancer or client-side beats the service's self-report, which misses the failures where the service never answered.

2. **Define "good" precisely, in a spec.** Which endpoints count? Do 4xx count against you (usually no, except 429/timeouts)? Retried requests counted once or twice? Health checks excluded? Write the exact query. Ambiguity here becomes an argument during the first bad week.

3. **Set the number from measured baseline, minus ambition, bounded by cost.** Measure 4+ weeks of actuals. If the journey currently delivers 99.7%, an SLO of 99.9% is a project commitment, not a target — either fund the project or set 99.5% honestly. Each added nine multiplies cost (redundancy, on-call, release friction) roughly 10×; check dependency ceilings too: you cannot offer 99.99% on top of a single dependency offering 99.9%.

4. **Make the SLO stricter than the SLA, if an SLA exists.** SLA = external contract with penalties; SLO = internal target that pages before the contract burns. Typical gap: SLA 99.5%, SLO 99.9%. No SLA? The SLO is still worth having — it's the definition of "reliable enough to stop investing."

5. **Derive the error budget and — the actual point — attach policy to it.** Budget = 1 − SLO over the window (99.9% / 30d ≈ 43 min of full downtime). Write the policy *before* it's needed:
   - Budget healthy → ship fast, take risks, spend it deliberately (chaos drills, risky migrations).
   - Budget burning fast → page (`alerting-design` step 3).
   - Budget exhausted → feature freeze / reliability-only sprints until back in budget, agreed with product *now*, not negotiated mid-crisis.
   An error budget nobody acts on is a dashboard decoration.

6. **Choose rolling windows (28/30d) over calendar months** — calendar resets create a "budget refill day" that invites gambling and makes month-boundary incidents weirdly cheap. Rolling windows keep the pressure smooth.

7. **Review quarterly:** SLO chronically overachieved by a wide margin → either tighten it or *deliberately spend* the slack (faster releases), because users are silently baselining on the delivered level (Hyrum's Law for reliability); chronically missed → fund the gap or lower the target honestly. An SLO that never changes anything gets deleted — three good SLOs beat fifteen ornamental ones.

## Common pitfalls
- SLOs on component metrics (CPU, pod restarts, cache hit rate). Users don't experience your CPU; they experience the journey. Components inform debugging, not objectives.
- 99.99% chosen because it sounds professional, on a team with business-hours on-call and single-region deploys. The math: 4.3 min/month of allowed downtime — one bad deploy spends a quarter's budget. Targets are commitments, price them.
- Measuring availability from the service's own logs — the outage where the LB never reached you scores as 100%.
- Latency SLO on the average, or on p99-as-a-number without a ratio framing — unmeasurable against a budget. "X% of requests under Yms" composes; "p99 < Y" doesn't.
- Error budget with no pre-agreed freeze policy: the first exhaustion becomes a product-vs-SRE standoff, resolved by whoever shouts best.
- Excluding "planned maintenance" generously until the SLO measures nothing users care about.

## Example
Search service, no targets, chronic reliability arguments. Journey: "query returns results." SLIs chosen: availability (non-5xx, timeouts count as bad, measured at the LB) and latency (fraction of queries < 800ms). Baseline over 6 weeks: 99.92% avail, 98.1% under 800ms. Set: 99.9% availability, 97% latency, 30d rolling — achievable, slightly ambitious on latency. Budget policy signed with product: fast-burn pages, exhaustion = freeze. Two months in, a bad index deploy burned 60% of budget in a day → page fired at 40× burn, rollback in 12 min, and the pre-agreed policy turned "should we still ship Friday's release?" into a lookup instead of a fight. Quarterly review tightened latency to 98% after the shard fix landed.

## Related skills
- `alerting-design` — burn-rate alerts defending these targets.
- `capacity-planning` — the headroom side of meeting the number.
- `metric-definition` — the same rigor applied to business metrics.
