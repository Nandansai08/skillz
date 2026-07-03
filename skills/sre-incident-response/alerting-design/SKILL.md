---
name: alerting-design
description: >
  Use when creating, tuning, or pruning alerts — symptom-vs-cause alerting,
  page-worthy vs ticket-worthy, thresholds, reducing pager noise. Triggers:
  "set up alerts for", "too many alerts", "alert fatigue", "should this
  page", "tune this alert", "we missed the outage because no alert fired".
---

# Alerting Design

## When to use this skill
- Adding alerts for a new service, or auditing a noisy pager.
- A postmortem showed detection was slow or missing.
- NOT for choosing what to measure in general — see `slo-definition` for the objectives alerts defend.

## Prerequisites
- The service's SLOs or at least its critical user journeys ("users must be able to X").
- Access to alert history for tuning work (what fired, what was actioned).

## Workflow

1. **Alert on symptoms, page on user pain.** The paging layer watches what users experience: error rate on the journey, latency percentiles, availability of the flow. Causes (CPU high, disk filling, pod restarts) become *ticket* alerts or dashboard context — a cause alert pages you for problems users never see, and misses problems whose cause you didn't predict. One symptom alert covers a thousand causes.

2. **Route every alert into exactly one of three lanes:**
   - **Page** (wake a human): user-facing symptom, needs action within minutes, and a human *can* act. All three or it doesn't page.
   - **Ticket** (business hours): needs action within days — disk 70%, cert expiring in 2 weeks, error budget burn elevated.
   - **Dashboard/log only:** context for debugging, no notification. Most cause-metrics live here.
   The audit question for every existing pager alert: "what did the responder *do* last five times this fired?" Answer "acknowledged it" = demote it.

3. **Set thresholds from SLOs via burn rate, not vibes.** Multi-window burn-rate alerting is the standard: page when the error budget is burning fast enough to matter (e.g., 14.4× burn over 1h AND 5m — the short window prevents alerting on a recovered spike), ticket on slow burn (3× over 24h). This ties every page to "we will miss the SLO if nobody acts," which is the definition of page-worthy.

4. **Make every alert carry its context:** the alert message includes what's broken *in user terms*, a dashboard link scoped to the right time range, the runbook link (`runbook-authoring`), and severity. A bare "HighErrorRate on prod-api-7" costs the responder ten minutes of orientation at 3am.

5. **Engineer against flapping and storms:** `for:` durations so transient blips don't page (but keep the paging window short enough for real outages — 2–5 min typical); grouping/inhibition so one dead database pages once, not once per dependent service (alertmanager inhibition rules, dependency-aware routing); and a global "too many alerts firing" meta-signal that pages *instead of* fifty individual pages.

6. **Test alerts like code.** Fire them synthetically (kill a canary instance, inject errors in staging, use `amtool`/unit tests on rules) — an alert that has never fired is untested detection. Include the *absence* case: a dead-man's-switch alert that fires when the metrics pipeline itself stops reporting, or every other alert goes quiet along with the outage.

7. **Review the pager monthly with three numbers:** pages per on-call week (sustainable: low single digits/night ~zero), actionable rate (pages that led to action — target >80%), and postmortem detection gaps (outages no alert caught). Noisy alerts get tuned or demoted *that week*; every gap gets a new symptom alert. This loop is what keeps the pager trusted.

## Common pitfalls
- Paging on every cause metric "for visibility." Sixty cause alerts produce alert fatigue, and fatigue produces the ignored page during the real outage — noise is not conservative, it's dangerous.
- Static thresholds on cyclical metrics: "traffic < 1000 rpm" fires every night at 4am, or misses a 50% drop at peak. Burn rates and relative drops handle cycles.
- Averages in alert conditions. Average latency hides the broken p99 subset (`production-debugging` step 4); alert on percentiles.
- Alerts created during incidents and never revisited — the pager becomes an archaeology of past outages, each stratum firing on new false positives. The monthly review (step 7) is the excavation.
- No dead-man's switch: the metrics agent dies, everything looks green, the outage is detected by customers. Silence must be able to alarm.
- Threshold tuned to "when it usually fires falsely" instead of "when action is needed" — codifying the noise instead of removing it.

## Example
Team pager audit: 94 pages/month, 11% actionable. Applied: 40 cause alerts (CPU, restarts, queue depth) demoted to tickets/dashboards; 4 symptom alerts built on the two SLOs (checkout availability, search latency) with 1h/5m + 24h burn-rate pairs; inhibition rule so DB-down suppresses its 12 dependents; dead-man's switch on the Prometheus remote-write. Next month: 9 pages, 89% actionable — and a real 6-minute checkout degradation paged in 3 minutes, previously invisible under the noise floor. One detection gap since (webhook delays) → one new symptom alert per the monthly loop.

## Related skills
- `slo-definition` — the objectives step 3's burn rates defend.
- `runbook-authoring` — what step 4's runbook link points to.
- `blameless-postmortem` — the source of detection-gap findings feeding step 7.
