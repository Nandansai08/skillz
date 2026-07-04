---
name: metric-definition
description: >
  Use when defining a metric people will act on — precise numerator and
  denominator, gaming resistance, edge-case policy, one owned definition.
  Triggers: "define a metric for", "what should we measure", "our numbers
  don't match", "north star metric", "two dashboards disagree", "how do
  we measure success". NOT for choosing statistical tests on the metric
  (see ab-test-analysis) — this is making the metric itself trustworthy.
---

# Metric Definition

## Overview

Most metric fights are denominator fights, and most dashboard discrepancies are unpinned nouns. A metric is a sentence with every noun pinned, its edge cases legislated, a guardrail paired against gaming, and one canonical implementation with an owner.

## When to Use

- Creating a metric for a dashboard, OKR, experiment, or exec report.
- Two teams' "same" metric disagrees and it's arbitration time.

**When NOT to use:**
- Experiment statistics on an existing metric — `ab-test-analysis`.

## Prerequisites

- The decision the metric will drive ("we'll change X if this moves") — a metric with no decision attached is a vanity number; establish this first or decline the request.

## The Workflow

1. **Write the metric as a sentence with every noun pinned.** Template: *[aggregation] of [precisely defined event/entity] per [precisely defined population] over [window], measured at [time anchor]*. "Weekly active users" becomes: "count of distinct user_ids with ≥1 *billable API call or dashboard session ≥30s* (not: login pings, health checks) in the 7 days ending Sunday UTC, among users whose account status was `active` at window end." Every vague noun is a future discrepancy.

2. **Interrogate the denominator harder than the numerator.** Most metric fights are denominator fights: conversion per *visitor, session, or signup*? Churn per *customers at period start, or average over period*? The denominator also carries the gaming surface — cutting it (reclassify inactive users) raises the rate without improving anything. Rule: the denominator's inclusion criteria must be immune to the actions of whoever's measured by the metric.

3. **Legislate the edge cases now, in writing:** refunds (net or gross, booked or refund date?), timezone (UTC vs user-local moves daily numbers ±5%), late-arriving events (restate with a note, or freeze at T+2d — never "whatever the pipeline does"), deleted/merged accounts, internal/test traffic (excluded, and the exclusion itself defined), currency conversion date. Each unlegislated edge becomes a "why did the number change" thread.

4. **Red-team it for gaming and Goodhart.** Ask: if a team were paid on this, what's the cheapest way to move it without creating the value it represents? Sales on bookings → bad-fit deals; support on tickets-closed → premature closes; engineering on coverage → assertion-free tests. Then pair the target with a **guardrail metric** that gaming would damage (bookings + 90-day retention; tickets-closed + reopen rate). A KPI shipped without its guardrail is an incentive bug.

5. **Match the metric's shape to the behavior's shape:** ratios normalize for volume but hide magnitude (report the numerator alongside — "conversion improved" while traffic halved is the classic); averages on skewed distributions lie (median/percentiles — `exploratory-data-analysis` step 3); leading vs lagging chosen deliberately (activation leads revenue by months — steer with the leading one, verify with the lagging one).

6. **Give it one canonical implementation and an owner.** The definition lives in a metrics/semantic layer (dbt metrics, LookML, or one blessed SQL view) that every dashboard reads — not re-implemented per-dashboard, which is where "two dashboards disagree" breeds. Owner's job: approve definition changes, version them (`revenue_v2` with a change log and a dual-reported transition month — silent redefinitions poison every historical comparison).

7. **Validate before shipping:** backfill 12 months and eyeball the trend against known history (launches, outages, seasonality should be visible — a metric that doesn't show the March incident isn't measuring what you think); reconcile against an independent source (finance's revenue, billing's count) and explain any gap; socialize the one-line definition WITH the number wherever displayed (a tooltip carrying the step-1 sentence).

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "Everyone knows what 'active user' means" | The MAU that counts token refreshes measures TTL config, not usage — and three teams 'know' three different definitions. The pinned sentence is what 'everyone knows' looks like written down. |
| "We'll tighten the definition once we see the data" | Definitions tuned after seeing numbers get tuned TOWARD the numbers someone wants. Pin first; the discipline is the point. |
| "A guardrail metric doubles the reporting work" | The unguarded target invites the gaming that costs a quarter to discover: tickets closed prematurely, coverage without assertions. One paired metric is the cheapest anti-Goodhart device known. |
| "Just exclude that weird edge case from the query" | Undocumented exclusions are how 'the number changed' threads start, and how audits end badly. Legislate in the definition doc or leave it in. |
| "Each dashboard computing it locally is fine — same SQL" | 'Same' SQL diverges at the third copy-paste; the semantic layer exists because it always does. One implementation or eventual arbitration. |
| "This metric would be interesting to track" | Interesting-with-no-decision is namespace pollution and maintenance load. The prerequisite question — what decision changes? — filters the wishlist. |

## Red Flags

- Two dashboards showing different values for the "same" metric.
- A metric definition living only in a query someone wrote once.
- Ratio reported without its numerator anywhere in sight.
- History restating silently as late data lands — screenshots disagreeing with dashboards.
- A target metric bonused/OKR'd with no guardrail pair.
- Definitions changed mid-quarter with no version marker.

## Verification

- [ ] The pinned sentence written, with aggregation/event/population/window/anchor all explicit — doc linked.
- [ ] Denominator immunity checked: whose actions can shrink it, and is that gameable? — noted.
- [ ] Edge-case ledger complete (refunds, timezone, late data, deletions, internal traffic, FX) — each with a ruling.
- [ ] Guardrail metric named and shipped alongside — dashboard shows the pair.
- [ ] One canonical implementation in the metrics layer — link; consuming dashboards enumerated.
- [ ] 12-month backfill eyeballed against known events; independent-source reconciliation within explained tolerance — both attached.

## Example

Ask: "define activation for the PLG funnel." Decision it drives: onboarding-flow investment. Sentence produced: "% of new signups (excl. internal domains + dupe emails) who create ≥1 project AND invite ≥1 teammate within 14 days of signup, cohorted by signup week UTC, frozen at day 15." Denominator red-team: excluding "signups who never verified email" was rejected — onboarding *owns* verification, so excluding it lets the team improve the number by ignoring their own funnel step. Guardrail: week-4 retention of activated users (guards against "activation theater" — hollow checklist prompts that inflate the action without the habit). Backfill validation surfaced that the mobile app never emitted `invite_sent` — fixed before launch, not after a quarter of wrong numbers. Definition shipped in the dbt semantic layer with the tooltip; the two pre-existing "activation" dashboards were deprecated the same week.

## Related skills

- `ab-test-analysis` — experimenting on metrics defined this way.
- `kpi-reporting-pack` — presenting these metrics to stakeholders.
- `slo-definition` — the same discipline for reliability metrics.
- `cohort-retention-analysis` — the guardrail metric's usual home.
