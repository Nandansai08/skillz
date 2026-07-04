---
name: eval-loop-for-agents
description: >
  Use when an agent's behavior needs measuring over time — building a
  test/eval harness that runs the agent repeatedly against a task set and
  scores drift or regression across prompt, model, and tool changes.
  Triggers: "eval my agent", "did the prompt change make it worse", "agent
  regression testing", "benchmark the agent", "model upgrade broke the
  agent", "score agent runs". NOT for evaluating a single run's failure
  (see tool-use-loop-debugging) or designing the per-run stop condition
  (see convergence-criteria-design).
---

# Eval Loop for Agents

## Overview

An agent without an eval harness changes blind: every prompt tweak, model upgrade, and tool swap is a bet with no scoreboard. This skill builds the harness — task set, scoring, variance handling, regression gating — so agent changes ship with evidence instead of vibes.

## When to Use

- Before shipping any change to an agent's prompt, model, tools, or loop structure.
- A model deprecation forces migration and "does it still work?" needs a real answer.
- The agent's quality is disputed ("it got worse lately") and nobody has numbers.
- Harvesting production failures into a permanent regression suite.

**When NOT to use:**
- Debugging why one specific run failed — that's `tool-use-loop-debugging`; evals measure populations, not incidents.
- Defining when a single run should stop — that's `convergence-criteria-design` (though the two share judge machinery).
- Evaluating a plain prompt template with deterministic output — snapshot tests suffice; this skill earns its cost when loops and tools make runs nondeterministic.

## Prerequisites

- Traced runs to harvest from (`feedback-loop-instrumentation` — its failure store is the seed corpus).
- A definition of task success per task type, even rough — the harness automates judgment; it can't invent it.

## The Workflow

1. **Build the task set from reality, in three tiers.** Harvest: (a) real production tasks with known-good outcomes (anonymized), (b) every debugged failure as a permanent case (`regression-test-from-bug`'s move at agent scale — the perseveration case, the drift case, each pinned), (c) synthetic edge cases probing known weak spots. Start small and honest: 20–50 tasks beats 500 unlabeled ones. Each task carries: input, success criteria (checkable where possible), and difficulty/category tags — the tags are what turn a score into a diagnosis later.

2. **Score with the checkable-first hierarchy.** Same ladder as `convergence-criteria-design` step 1: binary checks (task completed? output validates? tests pass?) > thresholds (cost, iterations, latency) > rubric'd LLM-judge for the subjective remainder. Per task, record BOTH outcome scores and trajectory metrics (iterations used, tokens, tool errors, budget exits) — two prompts with equal success rates and 2× different costs are not equal.

3. **Run each task multiple times — variance is the ambush.** Agents are nondeterministic; a single run per task measures luck.

   ```
   How many runs per task?
        |
        v
   pass rate near 0% or 100%?  --yes-->  3 runs confirms
        |no
        v
   decision-relevant task (gating a ship call)?
        |yes                    |no
        v                       v
   5-10 runs, report          3-5 runs,
   pass@k + variance          track trend only
   ```

   Report pass-rate with run counts ("7/10"), never bare percentages from n=1. A "regression" inside one task's normal variance band is noise wearing a headline — establish each task's band from baseline runs before trusting any delta.

4. **Calibrate the judge before trusting it.** Where LLM-judges score: written rubric with anchored examples, judge blind to which variant produced the output, and agreement checked against human labels on a 30–60 sample (report the agreement rate; below ~80%, fix the rubric before using the judge). Re-calibrate when the JUDGE's model changes — a judge upgrade can move every score with zero change in the agent. Warning: never let the judge model default to the same model the agent runs — self-preference bias inflates exactly the comparisons you care about.

5. **Baseline, then gate changes on deltas.** Freeze the current config's scores as baseline (config fingerprint from `feedback-loop-instrumentation` step 1: model, prompt version, tool versions). Every candidate change runs the same suite; ship rules pre-agreed: e.g., "ship if overall pass-rate within noise band AND no category regresses >X% AND cost within budget." Category-level readouts are the payload — the prompt change that lifts overall 2% while tanking the hardest category 20% is invisible in the aggregate.

6. **Wire it into the change process.** Cheap smoke tier (5–10 fast tasks) on every PR touching prompts/tools; full suite nightly or pre-release; results dashboarded over time so drift shows as a trend, not a surprise (`ci-pipeline-design` step 5's trigger-routing, eval edition). The suite that requires manual invocation gets invoked the week before the incident, not after.

7. **Feed and prune the suite continuously.** New production failure → new pinned case (with its trace). Tasks at 100% pass for months → candidates for the smoke tier or retirement (they've become regression insurance, not signal). Task set version-controlled and changelogged — comparing scores across different task sets is comparing nothing, and the silent task-set edit is this domain's silently-redefined metric (`metric-definition` step 6).

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "I tested the new prompt on a few examples and it looks better" | Hand-picked examples + single runs measure luck and confirmation bias. The 20-task suite with 3 runs each costs one coffee's worth of tokens and produces an actual answer. |
| "Evals are a big project — we'll build them later" | The 20-case v1 harness is an afternoon. "Later" means every change until then ships unmeasured, and the eventual harness starts with zero baseline history. |
| "The model upgrade is from the provider; it's surely better" | Provider-better on aggregate benchmarks routinely means worse on YOUR task distribution. Migrations are exactly when the suite earns its existence. |
| "Our tasks are too subjective to score" | Decompose: completion, constraint adherence, cost, and format are checkable on almost any task; the judged remainder is smaller than claimed (convergence-criteria-design step 1). |
| "We can't afford to run everything 5 times" | Then run 3, and report variance honestly. What's unaffordable is shipping a regression to production because n=1 said fine. |
| "The judge is an LLM anyway, so scores are meaningless" | An uncalibrated judge is meaningless. A rubric'd judge at 85% human-agreement is a measurement instrument with a known error bar — which is what every real instrument is. |

## Red Flags

- A prompt/model/tool change merged with no eval run attached.
- Scores quoted without run counts or task-set version ("it's at 80%" — of what, run how many times, on which suite?).
- The task set unchanged for months while production failures accumulated unharvested.
- Judge and agent sharing a model, unflagged.
- Overall score reported, category breakdown never — the aggregate hiding the tail.
- Baseline re-run and "adjusted" after a bad candidate result.

## Verification

- [ ] Task set exists in version control with success criteria and tags per task — link.
- [ ] Baseline scores recorded with config fingerprint and run counts — link to the run artifact.
- [ ] Judge (if any) has a written rubric and a measured human-agreement rate on a labeled sample.
- [ ] At least one real production failure is pinned as a suite case with its trace.
- [ ] A change was gated by the suite at least once — the PR/decision linked (the harness that has never gated anything is decoration).
- [ ] Per-category readout produced, not just the aggregate — artifact attached.

## Example

Team migrating a ticket-triage agent off a deprecating model. Suite built in two days: 42 tasks (28 harvested from traced production runs incl. 9 pinned past failures, 14 synthetic edge cases), scored by binary routing-correctness + escalation-precision judge (rubric'd, 88% agreement vs the support lead's labels, judge on a different model family). Baseline: 5 runs/task on the old model — which immediately paid off by revealing two tasks with 40–60% variance bands (flaky by nature; their deltas got wider gates). Candidate model: overall pass 84% vs baseline 82% — ship, per the aggregate. Category readout said otherwise: the "multi-issue tickets" tag dropped 71%→48%, outside its band. Diagnosis from the trajectory metrics: the new model exited two iterations earlier on average — under-decomposing compound tickets. Fixed with a prompt adjustment, re-ran: 86% overall, no category below band, cost −18%. Shipped with the eval artifact linked in the migration PR. The suite's nightly run caught one drift event the following quarter (a tool API change, not a model change) three days before the support team would have noticed it in their metrics.

## Related skills

- `feedback-loop-instrumentation` — supplies the traces and failure corpus.
- `convergence-criteria-design` — shared scoring hierarchy and judge calibration.
- `tool-use-loop-debugging` — produces the pinned regression cases.
- `ab-test-analysis` — the statistics discipline for variance and deltas.
- `regression-test-from-bug` — the same pin-every-failure philosophy, code edition.
