---
name: slo-definition
description: >
  Use when defining service reliability targets — choosing SLIs, setting
  SLO numbers, running error budgets, deciding what reliability to
  promise. Triggers: "define SLOs", "what should our uptime target be",
  "error budget", "which SLIs", "how many nines", "reliability targets
  for this service". NOT for building the alerts on top (see
  alerting-design) — this produces what those alerts defend.
---

# SLO Definition

## Overview

An SLO is a target with a policy attached: what users can expect, measured where users are, with pre-agreed consequences when the budget burns. Targets set from aspiration instead of baselines, or budgets nobody acts on, are dashboard decorations with better branding.

## When to Use

- Establishing reliability targets for a service (new or existing).
- An error budget exists on paper but changes nothing about decisions.

**When NOT to use:**
- The alerting layer — `alerting-design` builds burn-rate alerts on what this skill produces.

## Prerequisites

- Telemetry good enough to *measure* candidate SLIs for a few weeks (you set targets from data, not aspiration).
- The service's critical user journeys, named.

## The Workflow

1. **Pick SLIs per user journey, not per component.** For each critical journey ("user searches and gets results"), choose 1–2 indicators from the standard menu: **availability** (good requests / total requests), **latency** (proportion of requests faster than a threshold — "99% of requests < 400ms" is measurable as a ratio; a bare "p99 < Y" doesn't compose against a budget), **freshness/correctness** for pipelines. Measure as close to the user as possible: load-balancer or client-side beats the service's self-report, which scores the never-answered request as 100%.

2. **Define "good" precisely, in a spec.** Which endpoints count? Do 4xx count against you (usually no, except 429/timeouts)? Retried requests counted once or twice? Health checks excluded? Write the exact query. Ambiguity here becomes an argument during the first bad week.

3. **Set the number from measured baseline, minus ambition, bounded by cost.** Measure 4+ weeks of actuals. If the journey currently delivers 99.7%, an SLO of 99.9% is a project commitment, not a target — either fund the project or set 99.5% honestly. Each added nine multiplies cost roughly 10×; check dependency ceilings too: you cannot offer 99.99% on top of a single dependency offering 99.9%.

4. **Make the SLO stricter than the SLA, if an SLA exists.** SLA = external contract with penalties; SLO = internal target that pages before the contract burns. Typical gap: SLA 99.5%, SLO 99.9%. No SLA? The SLO is still worth having — it's the definition of "reliable enough to stop investing."

5. **Derive the error budget and — the actual point — attach policy to it.** Budget = 1 − SLO over the window (99.9% / 30d ≈ 43 min of full downtime). Write the policy *before* it's needed:
   - Budget healthy → ship fast, take risks, spend it deliberately (chaos drills, risky migrations).
   - Budget burning fast → page (`alerting-design` step 3).
   - Budget exhausted → feature freeze / reliability-only sprints until back in budget, agreed with product *now*, not negotiated mid-crisis.

6. **Choose rolling windows (28/30d) over calendar months** — calendar resets create a "budget refill day" that invites gambling and makes month-boundary incidents weirdly cheap. Rolling windows keep the pressure smooth.

7. **Review quarterly:** SLO chronically overachieved by a wide margin → either tighten it or *deliberately spend* the slack (faster releases), because users are silently baselining on the delivered level; chronically missed → fund the gap or lower the target honestly. An SLO that never changes anything gets deleted — three good SLOs beat fifteen ornamental ones.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "Four nines sounds like the professional target" | 99.99% is 4.3 min/month — one bad deploy spends a quarter's budget, and it demands multi-region + 24/7 on-call the team doesn't have. Targets are commitments; price them before promising them. |
| "Our service logs say we're at 99.98%" | Self-reported availability scores the outage where the LB never reached you as perfect uptime. Measure at the user's side of the boundary or the number is self-portraiture. |
| "We'll define the freeze policy when the budget actually runs out" | Exhaustion week is a product-vs-reliability standoff decided by whoever shouts best. The policy signed in peacetime is a lookup; in wartime it's a war. |
| "Excluding planned maintenance keeps the number clean" | Generous exclusions converge on an SLO that measures nothing users experience. Users don't experience your maintenance calendar as different downtime. |
| "Calendar months are simpler to report" | And the Jan-31 incident costs one day of budget while the Feb-1 incident costs a month's — same outage, tenfold difference. Rolling windows remove the arbitrage. |
| "More SLOs = more rigor — let's cover every endpoint" | Fifteen SLOs mean none of them gate anything. Rigor is three journey SLOs with policies that actually fire. |

## Red Flags

- SLOs on CPU, pod restarts, or cache hit rates (components, not journeys).
- The "good request" definition undocumented; two dashboards computing it differently.
- A target with no baseline measurement behind it.
- Error budget exhausted with zero process change — the policy that never fired.
- Availability measured from the service's own logs only.
- SLO overachieved by 10× for a year, untouched (slack unspent, users re-baselining).

## Verification

- [ ] Each SLO maps to a named user journey — the journey list linked.
- [ ] "Good" spec written as an exact query — linked; both dashboards use it.
- [ ] Target justified against ≥4 weeks of baseline — the measurement linked.
- [ ] Budget policy signed by product AND engineering, before exhaustion — doc linked.
- [ ] Rolling window configured — dashboard shows it.
- [ ] Quarterly review scheduled; last review's tighten/spend/delete decisions recorded.

## Example

Search service, no targets, chronic reliability arguments. Journey: "query returns results." SLIs chosen: availability (non-5xx, timeouts count as bad, measured at the LB) and latency (fraction of queries < 800ms). Baseline over 6 weeks: 99.92% avail, 98.1% under 800ms. Set: 99.9% availability, 97% latency, 30d rolling — achievable, slightly ambitious on latency. Budget policy signed with product: fast-burn pages, exhaustion = freeze. Two months in, a bad index deploy burned 60% of budget in a day → page fired at 40× burn, rollback in 12 min, and the pre-agreed policy turned "should we still ship Friday's release?" into a lookup instead of a fight. Quarterly review tightened latency to 98% after the shard fix landed.

## Related skills

- `alerting-design` — burn-rate alerts defending these targets.
- `capacity-planning` — the headroom side of meeting the number.
- `metric-definition` — the same rigor applied to business metrics.
