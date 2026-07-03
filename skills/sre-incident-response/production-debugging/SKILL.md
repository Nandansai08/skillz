---
name: production-debugging
description: >
  Use when diagnosing a live system's misbehavior through logs, metrics, and
  traces — without a local repro and without making production worse.
  Triggers: "debug this in production", "errors in prod but can't reproduce",
  "latency spike", "what's causing these 500s", "reading traces",
  "memory climbing in prod".
---

# Production Debugging

## When to use this skill
- A production symptom needs a cause: error spikes, latency, resource leaks, wrong outputs.
- Post-mitigation root-cause hunting after an incident.
- NOT for the first minutes of an active outage — run `incident-triage` first; mitigate, then come back here.

## Prerequisites
- Access to logs, metrics, traces, and deploy/change history for the affected services.
- Read-only discipline: agreement on what you may touch (and change-tracking for anything you do).

## Workflow

1. **Anchor on "what changed at T?"** Symptoms have start times. Line up the symptom's onset against: deploys (all services, not just the suspect), config/flag changes, dependency deploys, traffic pattern shifts, cron schedules, certificate expiries, data growth crossing a threshold. Most production mysteries are a change wearing a disguise; the ones that aren't are usually a *threshold* (disk, pool, integer) crossed gradually.

2. **Characterize the failure's shape before hypothesizing.** Slice the errored requests by every dimension the telemetry has: endpoint, region, instance, customer, user-agent, payload size. "All 500s are on POST /orders, from one AZ, on pods started after 14:00" *is* most of the diagnosis. Uniform failure → shared dependency; partitioned failure → the partition names the suspect.

3. **Walk the funnel: metrics → logs → traces.** Metrics say *where* (which service's latency broke first — check upstream vs downstream to avoid blaming the victim); logs say *what* (the actual exception, the actual query); traces say *how* (which hop in this specific slow request ate the time). Grab exemplar request IDs from the symptom window and follow the same request through all three.

4. **Read latency as a distribution, never an average.** p50 flat + p99 exploding = a subset is hit (lock contention, GC pauses, one slow shard, retry storms) — averages hide this entirely. Bimodal histograms mean two populations; find what distinguishes them (step 2's slicing).

5. **Form one hypothesis and try to *disprove* it cheaply.** State it falsifiably: "pool exhaustion — then active connections should be pinned at max during spikes." Check the one metric. Wrong? Next hypothesis. This loop beats the shotgun ("restart it, bump the memory, clear the cache") that destroys the evidence and sometimes fixes it without telling you what it was.

6. **Escalate observation tools carefully, in increasing invasiveness:** targeted log-level increase on one instance → capture a heap/goroutine/thread dump on one instance out of the LB → CPU/memory profiler (continuous profiling if you have it) → tcpdump/eBPF as last resort. One instance, time-boxed, announced in the incident/team channel — profilers and debug logging have themselves caused outages.

7. **Confirm the mechanism end-to-end before closing.** The cause must explain the shape from step 2 (why that AZ? why only new pods?), the timing from step 1, and the recovery. A fix that works for unexplained reasons is a coincidence with good PR. Then feed it back: the regression test (`regression-test-from-bug`), the missing alert (`alerting-design`), the runbook entry.

## Common pitfalls
- Restarting the sick instance first. It relieves the symptom and cremates the evidence (heap state, connection tables, thread stacks). Capture the dump, *then* restart.
- Trusting logs over reality: "no errors in the logs" often means errors are swallowed (`error-handling-strategy` step 7) or the pod OOM-killed before flushing. Absence of logs is a clue, not an acquittal.
- Blaming the top of the stack trace / the service where the alert fired. Timeouts cascade upstream; the first-failing component is often three hops down. Trace the exemplar.
- Correlation-jumping: "it started when we deployed X" — deploys also shift traffic, warm caches cold, and reset connections. Step 7's mechanism check separates the true cause from the co-traveler.
- Debugging on the whole fleet: log level to DEBUG everywhere melts the log pipeline and the disks. One instance.

## Example
Symptom: p99 on /search 8× since ~03:00, p50 flat. Step 1: no deploys; but a nightly reindex cron finished 02:58. Step 2: slow requests all hit queries with >2 terms. Step 3: traces show time in Elasticsearch, one shard. Hypothesis (falsifiable): reindex left one hot shard with a pathological segment. Check: that shard's segment count 40× siblings — confirmed without touching anything. Mitigation: force-merge the shard (announced, one shard first). p99 recovered; mechanism explained shape (multi-term queries fan out to all shards, so one bad shard gates them) and timing. Follow-ups: post-reindex segment-count alert, force-merge added to the reindex job.

## Related skills
- `incident-triage` — the wrapper when this is happening during an outage.
- `feedback-loop-instrumentation` — making systems this debuggable before you need it.
- `debugging-by-bisection` — when there IS a repro, bisect instead.
- `alerting-design` — turning step 7's findings into earlier detection.
