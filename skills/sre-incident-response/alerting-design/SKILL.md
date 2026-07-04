---
name: alerting-design
description: >
  Use when creating, tuning, or pruning alerts — symptom-vs-cause
  alerting, page-worthy vs ticket-worthy, thresholds, reducing pager
  noise. Triggers: "set up alerts for", "too many alerts", "alert
  fatigue", "should this page", "tune this alert", "we missed the outage
  because no alert fired". NOT for choosing what to measure in general —
  see slo-definition for the objectives alerts defend.
---

# Alerting Design

## Overview

A noisy pager is not conservative — it's dangerous: fatigue is what makes the real page get ignored. Symptom-based alerts tied to SLO burn rates, three routing lanes, and a monthly actionability review keep the pager trusted, which is the only property a pager needs.

## When to Use

- Adding alerts for a new service, or auditing a noisy pager.
- A postmortem showed detection was slow or missing.

**When NOT to use:**
- Defining the reliability objectives themselves — `slo-definition` produces what these alerts defend.

## Prerequisites

- The service's SLOs or at least its critical user journeys ("users must be able to X").
- Access to alert history for tuning work (what fired, what was actioned).

## The Workflow

1. **Alert on symptoms, page on user pain.** The paging layer watches what users experience: error rate on the journey, latency percentiles, availability of the flow. Causes (CPU high, disk filling, pod restarts) become *ticket* alerts or dashboard context — a cause alert pages you for problems users never see, and misses problems whose cause you didn't predict. One symptom alert covers a thousand causes.

2. **Route every alert into exactly one of three lanes:**
   - **Page** (wake a human): user-facing symptom, needs action within minutes, and a human *can* act. All three or it doesn't page.
   - **Ticket** (business hours): needs action within days — disk 70%, cert expiring in 2 weeks, error budget burn elevated.
   - **Dashboard/log only:** context for debugging, no notification.
   The audit question for every existing pager alert: "what did the responder *do* last five times this fired?" Answer "acknowledged it" = demote it.

3. **Set thresholds from SLOs via burn rate, not vibes.** Multi-window burn-rate alerting is the standard: page when the error budget is burning fast enough to matter (e.g., 14.4× burn over 1h AND 5m — the short window prevents alerting on a recovered spike), ticket on slow burn (3× over 24h). This ties every page to "we will miss the SLO if nobody acts," which is the definition of page-worthy. Warning: static thresholds on cyclical metrics fire every night at 4am or miss the 50% drop at peak — burn rates and relative drops handle cycles; and alert on percentiles, never averages.

4. **Make every alert carry its context:** the alert message includes what's broken *in user terms*, a dashboard link scoped to the right time range, the runbook link (`runbook-authoring`), and severity. A bare "HighErrorRate on prod-api-7" costs the responder ten minutes of orientation at 3am.

5. **Engineer against flapping and storms:** `for:` durations so transient blips don't page (but keep the paging window short enough for real outages — 2–5 min typical); grouping/inhibition so one dead database pages once, not once per dependent service; and a global "too many alerts firing" meta-signal that pages *instead of* fifty individual pages.

6. **Test alerts like code.** Fire them synthetically (kill a canary instance, inject errors in staging, unit-test the rules) — an alert that has never fired is untested detection. Include the *absence* case: a dead-man's-switch alert that fires when the metrics pipeline itself stops reporting — silence must be able to alarm.

7. **Review the pager monthly with three numbers:** pages per on-call week (sustainable: low single digits; nights ~zero), actionable rate (pages that led to action — target >80%), and postmortem detection gaps (outages no alert caught). Noisy alerts get tuned or demoted *that week*; every gap gets a new symptom alert. This loop is what keeps the pager trusted.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "Page on everything — better safe than sorry" | Noise is not safety. At 94 pages/month and 11% actionable, the one page that matters gets acknowledged on muscle memory. Fatigue is the mechanism by which over-alerting causes missed outages. |
| "CPU alerts give us early warning before users feel it" | They page for a thousand non-problems and miss the problem whose cause you didn't predict. Causes inform debugging (dashboard lane); symptoms define paging. |
| "We'll tune the noisy alert eventually" | Every week it stays, it trains responders to ignore its lane. 'That week' (step 7) is the rule because alert-trust decays faster than backlogs move. |
| "This alert has never fired — it's clearly not hurting anything" | Never-fired means never-tested: it may be wired to a dead metric or an impossible threshold. Synthetic firing (step 6) is the only proof of detection. |
| "The threshold is set where it stops false-alarming" | Tuned-to-silence codifies the noise level instead of the action level. The threshold question is 'when must a human act,' answered by burn rate, not by annoyance minimization. |
| "Everything's green, the metrics pipeline must be fine" | All-green is also what a dead metrics pipeline looks like. The dead-man's switch exists because silence and health are indistinguishable without it. |

## Red Flags

- Pages routinely acknowledged with no action taken.
- Alert rules with no runbook links, no dashboard links.
- On-call sleep interrupted by disk-percentage tickets.
- A single dependency failure producing dozens of simultaneous pages.
- No alert has ever been synthetically fired.
- Outage detected by customers while dashboards showed green (no dead-man's switch).

## Verification

- [ ] Every paging alert maps to a user-facing symptom and an SLO burn condition — rule annotations show it.
- [ ] Three-lane routing implemented; the demote-audit run on existing pages (actions-taken history reviewed) — results attached.
- [ ] Every page carries runbook + dashboard links — spot-check three rules.
- [ ] Inhibition/grouping configured for known dependency fan-outs — config linked.
- [ ] Each new alert fired synthetically at least once — test run linked.
- [ ] Dead-man's switch live — proof it fires when metrics stop (staging test).
- [ ] Monthly review scheduled with the three metrics dashboarded.

## Example

Team pager audit: 94 pages/month, 11% actionable. Applied: 40 cause alerts (CPU, restarts, queue depth) demoted to tickets/dashboards; 4 symptom alerts built on the two SLOs (checkout availability, search latency) with 1h/5m + 24h burn-rate pairs; inhibition rule so DB-down suppresses its 12 dependents; dead-man's switch on the Prometheus remote-write. Next month: 9 pages, 89% actionable — and a real 6-minute checkout degradation paged in 3 minutes, previously invisible under the noise floor. One detection gap since (webhook delays) → one new symptom alert per the monthly loop.

## Related skills

- `slo-definition` — the objectives step 3's burn rates defend.
- `runbook-authoring` — what step 4's runbook link points to.
- `blameless-postmortem` — the source of detection-gap findings feeding step 7.
