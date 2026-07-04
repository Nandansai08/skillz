---
name: feedback-loop-instrumentation
description: >
  Use when making an agent or iterative system's behavior observable —
  logging/tracing per iteration, run-level traces, the metrics that reveal
  drift and thrash over time. Triggers: "instrument my agent", "can't tell
  what the agent did", "agent observability", "trace agent runs", "log the
  loop", "why did this run cost so much".
---

# Feedback Loop Instrumentation

## When to use this skill
- Building an agent/iterative system and deciding what to record (ideally: before the first mystery, which is otherwise the trigger).
- A loop misbehaved and the honest answer to "what happened?" is "no idea" — instrument first, then return to `tool-use-loop-debugging` with evidence.
- NOT general service observability (`production-debugging`'s metrics/logs/traces funnel covers that) — loops add the iteration dimension: behavior ACROSS iterations and ACROSS runs is the payload here.

## Prerequisites
- The loop's design vocabulary (`agent-loop-design`'s phases, state schema, exits) — instrumentation names things; unnamed structure produces unnameable logs.

## Workflow

1. **Adopt the two-level trace model: run and iteration.** A **run** (one task execution) carries: run ID, task/goal, config fingerprint (model, prompt version, tool versions — the fields without which cross-run comparison is meaningless), start/end, exit reason (which exit fired — success/failure/budget/escalation), totals (iterations, tokens, cost, wall time). An **iteration** (one loop pass) carries: iteration number, phase, the plan stated, action taken (tool + args), observation received (result + truncation flag), reflection verdict, state-size snapshot, per-iteration cost. Standard tracing infrastructure fits (spans: run = parent, iterations = children — OTel or the LLM-specific platforms: LangSmith/Langfuse/Braintrust) — the schema matters more than the vendor.

2. **Record the agent's INPUTS faithfully, not just its outputs:** the exact context window content (or a reproducible pointer to it: prompt version + state snapshot + retrieved docs) per LLM call — because "why did it do that?" is answered by what it SAW, and reconstructed-context debugging is fiction (`tool-use-loop-debugging` steps 2–3 are unanswerable without this). Same fidelity for tool results as-returned vs as-shown (truncation and summarization are themselves events worth logging — the gap between them is a recurring root cause).

3. **Emit the loop-health metrics that reveal pathology as trends:**
   - **Progress proxy per iteration** (tests passing, TODOs closed, `convergence-criteria-design`'s metric) — flat = drift, oscillating = thrash, visible in one chart.
   - **Action-repetition rate** (the fingerprint hashes from `tool-use-loop-debugging` step 6, emitted as a metric) — rising = perseveration brewing.
   - **Iterations / tokens / cost per run**, distributioned (p50/p95 — the p95 run is where the money goes; `runaway-cost-guardrails` sets caps against these numbers).
   - **Exit-reason mix over time** — budget-exits rising means tasks outgrowing the loop or the loop degrading; the single best early-warning metric for fleet-level agent health.
   - **Per-tool error and latency rates** — the lying/flaky tool (`tool-use-loop-debugging` step 5) surfaces here before it surfaces as a mystery.

4. **Make single-run reading cheap: the run report.** Auto-render each run as a human-scannable narrative — goal, then per-iteration one-liners (`#4 plan: fix import | act: edit foo.py | obs: test still fails (same error) | reflect: trying new approach`), then exit + totals. The raw trace answers deep questions; the report answers "what happened here?" in 30 seconds — and whether anyone actually reads failures is determined by this artifact's existence (a trace no one can skim is write-only storage).

5. **Sample deliberately, keep failures fully:** full-fidelity traces (complete contexts) are heavy — sample happy-path runs (e.g., 10% full-fidelity, all runs at metadata level) but keep 100% of failures, budget-exits, and escalations at full fidelity (the failures ARE the debugging corpus and the future eval cases — `eval-loop-for-agents` step 2 harvests from exactly this store). Redact secrets/PII at capture, not at read (`secrets-management`'s logging-redaction rule — traces are logs with extra ambition).

6. **Wire the alerts on loop-shaped symptoms** (`alerting-design`'s lanes apply): page-worthy — cost-per-hour breach, budget-exit rate spike (the fleet is spinning), a tool's error rate cliff; ticket-worthy — p95 iteration counts creeping, repetition-rate trending up, progress-proxy degradation across versions. The pre-agent-era mistake to avoid: alerting only on service health (latency, 5xx) while the agents run "healthily" in circles — loop pathology is invisible to service-level monitors by construction.

7. **Close the loop on the instrumentation itself:** every debugging session that needed a missing field adds that field (the instrumentation grows from its own gaps — `blameless-postmortem`'s detect-faster action items, agent edition); every fixed pathology's signature becomes a metric or alert (the thrash you debugged once should be a dashboard line forever); and version the trace schema so cross-time comparisons survive the system's evolution (`metric-definition` step 6's same rule — silently changed semantics poison every trend).

## Common pitfalls
- Output-only logging: final answers recorded, the path to them discarded — the "why did it do that?" question unanswerable by design (step 2's inputs-fidelity is the difference between observability and a results ledger).
- Everything-at-full-fidelity: the trace store that costs more than the agents — then gets turned off entirely in the next cost review, restoring blindness. Sampling with failure-completeness (step 5) is the sustainable middle.
- Metrics without the iteration dimension: token totals and latency tracked, repetition and progress invisible — service-shaped monitoring of a loop-shaped system (step 3's list exists because loops fail in loop-shaped ways).
- The unreadable trace: everything captured, nothing skimmable — debugging sessions that start with an hour of JSON spelunking, so nobody starts them (step 4's report is adoption infrastructure, not polish).
- Reconstructing context instead of recording it: "the prompt was probably..." — the reproducibility gap that turns every investigation into archaeology (step 2, again, because it's the one most skipped).
- Instrumentation as a launch-week afterthought: the first production mystery arrives before the traces do — and gets debugged blind, expensively, exactly once before this skill gets invoked properly.

## Example
Team running a support-ticket-resolution agent, monthly cost mysteriously doubled, "what happened?" unanswerable — the system logged final resolutions only. Instrumentation retrofit per the schema: run/iteration spans in Langfuse, config fingerprints, full-context capture (sampled 10%, failures 100%), the run-report renderer, and the step-3 metric set. The mystery resolved within two days of data: exit-reason mix showed budget-exits at 31% (previously invisible — those runs had been silently eating 60% of tokens); the repetition-rate metric pointed at one tool — the knowledge-base search whose new "relevance" update returned near-empty results, sending agents into progressively desperate query reformulation (a perseveration variant: the tool was answering, uselessly). Fixes: tool rollback (its error rate had been zero — the metrics gap was result-QUALITY, so a result-emptiness metric got added per step 7), repetition detector wired to the new metric. Cost back to baseline in a week. The lasting payoff arrived at quarter's end: a prompt-version change showed a progress-proxy regression within 200 sampled runs — caught by the trend line, rolled back before the cost graph ever noticed (the instrumentation paying forward: the last incident's blindness became the next incident's 200-run early warning). The failure store, meanwhile, had accumulated 340 full-fidelity budget-exit traces — which seeded `eval-loop-for-agents`' first regression suite.

## Related skills
- `tool-use-loop-debugging` — the investigations this evidence enables.
- `agent-loop-design` — the structure that gives trace fields their names.
- `runaway-cost-guardrails` — caps set against these distributions.
- `eval-loop-for-agents` — harvests the failure store.
- `alerting-design` — the lane discipline for step 6.
