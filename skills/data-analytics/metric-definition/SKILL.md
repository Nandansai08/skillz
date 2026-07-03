---
name: metric-definition
description: >
  Use when defining a metric people will act on — precise numerator and
  denominator, gaming resistance, edge-case policy, one owned definition.
  Triggers: "define a metric for", "what should we measure", "our numbers
  don't match", "north star metric", "two dashboards disagree",
  "how do we measure success".
---

# Metric Definition

## When to use this skill
- Creating a metric for a dashboard, OKR, experiment, or exec report.
- Two teams' "same" metric disagrees and it's arbitration time.
- NOT for choosing statistical tests on the metric — that's `ab-test-analysis`; this is making the metric itself trustworthy.

## Prerequisites
- The decision the metric will drive ("we'll change X if this moves") — a metric with no decision attached is a vanity number; establish this first or decline the request.

## Workflow

1. **Write the metric as a sentence with every noun pinned.** Template: *[aggregation] of [precisely defined event/entity] per [precisely defined population] over [window], measured at [time anchor]*. "Weekly active users" becomes: "count of distinct user_ids with ≥1 *billable API call or dashboard session ≥30s* (not: login pings, health checks) in the 7 days ending Sunday UTC, among users whose account status was `active` at window end." Every vague noun in the sentence is a future dashboard discrepancy.

2. **Interrogate the denominator harder than the numerator.** Most metric fights are denominator fights: conversion rate per *visitor, session, or signup*? Churn per *customers at period start, or average over period*? Revenue per user including *free-tier users or not*? Denominators also carry the gaming surface — cutting the denominator (reclassify inactive users) raises the rate without improving anything. Rule: the denominator's inclusion criteria must be immune to the actions of whoever's measured by the metric.

3. **Legislate the edge cases now, in writing:** refunds and cancellations (net or gross, and *when* — booked date or refund date?), timezone (UTC vs user-local changes daily numbers by ±5%), late-arriving events (does yesterday's number restate? — pick "restate with a note" or "freeze at T+2d", never "whatever the pipeline does"), deleted/merged accounts, internal/test traffic (excluded, and the exclusion is itself defined), currency conversion date. Each unlegislated edge becomes a "why did the number change" thread.

4. **Red-team it for gaming and Goodhart.** Ask: if a team were paid on this, what's the cheapest way to move it without creating the value it represents? Sales paid on bookings → bad-fit deals; support on tickets-closed → premature closes; engineering on coverage → assertion-free tests (`coverage-analysis`). Then pair the target metric with a **guardrail metric** that gaming would damage (bookings + 90-day retention; tickets-closed + reopen rate; deploy frequency + change-failure rate). A KPI shipped without its guardrail is an incentive bug.

5. **Match the metric's shape to the behavior's shape:** ratios normalize for volume but hide magnitude (report the numerator alongside); averages on skewed distributions lie (median or percentiles — `exploratory-data-analysis` step 3); pick leading vs lagging deliberately (revenue lags product changes by months; activation leads it — dashboards need the leading one to steer, the lagging one to verify).

6. **Give it one canonical implementation and an owner.** The definition lives in a metrics layer / semantic layer (dbt metrics, LookML, or at minimum one blessed SQL view) that every dashboard reads — not re-implemented per-dashboard, which is where the "two dashboards disagree" pathology breeds. Owner's job: approve definition changes, version them (`revenue_v2` with a change log and a dual-reported transition month — silent redefinitions poison every historical comparison).

7. **Validate before shipping:** backfill 12 months and eyeball the trend against known history (launches, outages, seasonality should be visible — a metric that doesn't show the March incident isn't measuring what you think); reconcile against an independent source (finance's revenue, the billing system's count) and explain any gap to the percent; then socialize the one-line definition WITH the number wherever it's displayed (a tooltip carrying the sentence from step 1).

## Common pitfalls
- Ratio-only reporting: conversion "improved" from 2% to 4% because traffic (denominator) halved. Always show the pair.
- The unpinned "active": a metric named MAU that counts login-token refreshes measures token TTL config, not usage. The step-1 sentence exists for this.
- Restating history silently as late data arrives — every stakeholder screenshot now disagrees with the dashboard, and trust in ALL metrics drops. Legislate the restatement policy (step 3).
- Metric changed to "fix" a bad quarter. Versioning + owner + change log is the antibody; a definition that moves under pressure measures pressure.
- Twenty "key" metrics. Attention is the scarce resource; one decision-metric plus guardrails per surface. If everything is key, the dashboard is a mural.
- Defining without a decision attached — "interesting to know" metrics accumulate maintenance cost and pollute the namespace until nobody knows which numbers matter.

## Example
Ask: "define activation for the PLG funnel." Decision it drives: onboarding-flow investment. Sentence produced: "% of new signups (excl. internal domains + dupe emails) who create ≥1 project AND invite ≥1 teammate within 14 days of signup, cohorted by signup week UTC, frozen at day 15." Denominator red-team: excluding "signups who never verified email" was rejected — onboarding *owns* verification, so excluding it lets the team improve the number by ignoring their own funnel step. Guardrail: week-4 retention of activated users (guards against "activation theater" — hollow checklist prompts that inflate the action without the habit). Backfill validation surfaced that the mobile app never emitted `invite_sent` — fixed before launch, not after a quarter of wrong numbers. Definition shipped in the dbt semantic layer with the tooltip; the two pre-existing "activation" dashboards were deprecated the same week.

## Related skills
- `ab-test-analysis` — experimenting on metrics defined this way.
- `kpi-reporting-pack` — presenting these metrics to stakeholders.
- `slo-definition` — the same discipline for reliability metrics.
- `cohort-retention-analysis` — the guardrail metric's usual home.
