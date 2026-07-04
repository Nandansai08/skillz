---
name: production-debugging
description: >
  Use when diagnosing a live system's misbehavior through logs, metrics,
  and traces — without a local repro and without making production worse.
  Triggers: "debug this in production", "errors in prod but can't
  reproduce", "latency spike", "what's causing these 500s", "reading
  traces", "memory climbing in prod". NOT for the first minutes of an
  active outage (run incident-triage first; mitigate, then return here).
---

# Production Debugging

## Overview

Production mysteries yield to method: anchor on what changed, characterize the failure's shape before hypothesizing, walk metrics→logs→traces on the same exemplar request, and try to *disprove* one falsifiable hypothesis at a time. The shotgun (restart it, bump memory, clear cache) destroys the evidence and occasionally fixes it without telling you what it was.

## When to Use

- A production symptom needs a cause: error spikes, latency, resource leaks, wrong outputs.
- Post-mitigation root-cause hunting after an incident.

**When NOT to use:**
- Active outage, users down — `incident-triage` first; mitigate, then diagnose.
- A local repro exists — `debugging-by-bisection` is cheaper and safer.

## Prerequisites

- Access to logs, metrics, traces, and deploy/change history for the affected services.
- Read-only discipline: agreement on what you may touch (and change-tracking for anything you do).

## The Workflow

1. **Anchor on "what changed at T?"** Symptoms have start times. Line up the symptom's onset against: deploys (all services, not just the suspect), config/flag changes, dependency deploys, traffic pattern shifts, cron schedules, certificate expiries, data growth crossing a threshold. Most production mysteries are a change wearing a disguise; the ones that aren't are usually a *threshold* (disk, pool, integer) crossed gradually.

2. **Characterize the failure's shape before hypothesizing.** Slice the errored requests by every dimension the telemetry has: endpoint, region, instance, customer, user-agent, payload size. "All 500s are on POST /orders, from one AZ, on pods started after 14:00" *is* most of the diagnosis. Uniform failure → shared dependency; partitioned failure → the partition names the suspect.

3. **Walk the funnel: metrics → logs → traces.** Metrics say *where* (which service's latency broke first — check upstream vs downstream to avoid blaming the victim); logs say *what* (the actual exception, the actual query); traces say *how* (which hop in this specific slow request ate the time). Grab exemplar request IDs from the symptom window and follow the same request through all three. Warning: "no errors in the logs" often means errors are swallowed (`error-handling-strategy` step 7) or the pod died before flushing — absence of logs is a clue, not an acquittal.

4. **Read latency as a distribution, never an average.** p50 flat + p99 exploding = a subset is hit (lock contention, GC pauses, one slow shard, retry storms) — averages hide this entirely. Bimodal histograms mean two populations; find what distinguishes them (step 2's slicing).

5. **Form one hypothesis and try to *disprove* it cheaply.** State it falsifiably: "pool exhaustion — then active connections should be pinned at max during spikes." Check the one metric. Wrong? Next hypothesis. This loop beats the shotgun that destroys evidence.

6. **Escalate observation tools carefully, in increasing invasiveness:** targeted log-level increase on ONE instance → heap/goroutine/thread dump on one instance out of the LB → CPU/memory profiler → tcpdump/eBPF as last resort. One instance, time-boxed, announced — profilers and debug logging have themselves caused outages. And capture the dump BEFORE any restart: restarting the sick instance relieves the symptom and cremates the evidence.

7. **Confirm the mechanism end-to-end before closing.** The cause must explain the shape from step 2 (why that AZ? why only new pods?), the timing from step 1, and the recovery. A fix that works for unexplained reasons is a coincidence with good PR. Then feed it back: the regression test (`regression-test-from-bug`), the missing alert (`alerting-design`), the runbook entry.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "Restart it first, debug later" | The restart destroys heap state, connection tables, and thread stacks — the entire evidence locker. Dump first (one instance, two minutes), then restart. |
| "It started right after the deploy, so the deploy is the cause" | Deploys also shift traffic, cold caches, and reset connections — perfect co-travelers. The mechanism check (step 7) separates cause from coincidence before the wrong fix ships. |
| "The alert fired on service X, so the bug is in service X" | Timeouts cascade upstream; the first-failing component is often three hops down. Trace the exemplar before assigning blame. |
| "Average latency is fine, so it's a few users' network" | p50-flat-p99-exploding is the signature of a real subset problem (locks, GC, one shard). Averages are where tail incidents hide. |
| "Turn on debug logging everywhere so we don't miss anything" | Fleet-wide DEBUG melts the log pipeline and the disks — observation becoming the outage. One instance, time-boxed, announced. |
| "The fix worked, ship it — we can explain it later" | Unexplained recovery is a coincidence on a timer. If the mechanism doesn't explain the shape and timing, the real cause is still live. |

## Red Flags

- Multiple "fixes" applied in one window; recovery unattributable.
- The sick instance restarted before anyone captured state.
- Hypotheses tested by applying fixes to prod rather than by checking metrics.
- Diagnosis concluded without explaining the failure's partition (why only that AZ/customer/endpoint).
- Log absence treated as error absence.
- No regression test, alert, or runbook entry produced afterward.

## Verification

- [ ] Change-anchor table built: symptom onset vs deploys/config/cron/data events — in the investigation notes.
- [ ] Failure shape characterized: the slicing dimensions and the partition found — documented.
- [ ] Exemplar request followed through metrics, logs, AND traces — IDs noted.
- [ ] Each hypothesis stated falsifiably with its disproving check — the trail visible in notes.
- [ ] Mechanism explains shape + timing + recovery — one paragraph in the writeup.
- [ ] Feedback shipped: regression test, alert, or runbook entry linked.

## Example

Symptom: p99 on /search 8× since ~03:00, p50 flat. Step 1: no deploys; but a nightly reindex cron finished 02:58. Step 2: slow requests all hit queries with >2 terms. Step 3: traces show time in Elasticsearch, one shard. Hypothesis (falsifiable): reindex left one hot shard with a pathological segment. Check: that shard's segment count 40× siblings — confirmed without touching anything. Mitigation: force-merge the shard (announced, one shard first). p99 recovered; mechanism explained shape (multi-term queries fan out to all shards, so one bad shard gates them) and timing. Follow-ups: post-reindex segment-count alert, force-merge added to the reindex job.

## Related skills

- `incident-triage` — the wrapper when this is happening during an outage.
- `feedback-loop-instrumentation` — making systems this debuggable before you need it.
- `debugging-by-bisection` — when there IS a repro, bisect instead.
- `alerting-design` — turning step 7's findings into earlier detection.
