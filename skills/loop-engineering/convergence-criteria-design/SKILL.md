---
name: convergence-criteria-design
description: >
  Use when a loop has no natural stopping point — defining "done" for
  iterative refinement, generation-critique cycles, and optimization loops
  so they stop at good-enough instead of forever. Triggers: "when should
  the loop stop", "define done for the agent", "it keeps polishing
  forever", "good enough criteria", "diminishing returns in the loop",
  "stopping condition for refinement".
---

# Convergence Criteria Design

## When to use this skill
- A loop refines its own output (draft→critique→revise, optimize→measure→adjust) and could run indefinitely.
- An agent "completes" tasks either too early (declared done, wasn't) or never (polishing past all value).
- NOT for loops with natural binary exits (tests pass, the record exists) — those have their "done"; this skill serves the loops that DON'T, and feeds the success-exit slot in `agent-loop-design` step 3.

## Prerequisites
- Clarity on what the output is FOR — "good enough" is downstream-defined (good enough to ship? to hand a human editor? to feed the next pipeline stage?), and criteria designed without the consumer named are aesthetics.

## Workflow

1. **Fight for a checkable criterion before accepting a judged one — the hierarchy:**
   - **Objective-binary** (tests pass, schema validates, the build compiles, all references resolve): use it wherever it exists; cheap, ungameable, unambiguous.
   - **Objective-threshold** (error below ε, score above X, latency under budget): next best — the threshold needs justifying (step 3) but the measurement is mechanical.
   - **Judged** (LLM-judge rubric, human spot-check): the fallback for genuinely subjective quality — usable, with step 4's discipline.
   Most "unmeasurable" quality decomposes partially: a "well-written report" contains checkable parts (every claim sourced, all sections present, length in range, zero broken links) plus a judged remainder — extract the checkable majority FIRST; the judged surface shrinks to where judgment is genuinely needed.

2. **Add the delta-based stop for refinement loops — converging means CHANGE is dying:** track the improvement per iteration (score delta, diff size, critique-severity trend) and stop when marginal gain falls below marginal cost for K consecutive iterations (K=2–3 absorbs noise; single-iteration deltas lie). This is the diminishing-returns detector, and it catches what absolute thresholds miss: the loop that will NEVER hit 0.95 but plateaued at 0.91 three iterations ago should stop at the plateau, not the budget cap (`capacity-planning`'s knee-finding, applied to iteration value).

3. **Set thresholds from measured baselines, not aspiration:** run the loop on 20–30 representative cases, plot score-vs-iterations — the empirical plateau tells you what's achievable, where the knee sits, and what threshold separates "genuinely good" from "the loop's ceiling" (`slo-definition` step 3's identical discipline: targets from actuals, ambition priced separately). A threshold above the loop's empirical ceiling doesn't produce better outputs; it converts every run into a budget-exit (`runaway-cost-guardrails` catching what this skill misconfigured).

4. **If judging, engineer the judge like the component it is:** a written rubric with per-dimension anchors (what a 3 vs a 5 looks like, with examples — "rate the quality 1–10" is a mood ring); the judge sees the GOAL and the rubric, not the generator's reasoning (contamination inflates scores); calibrate judge-vs-human agreement on a labeled sample before trusting it (and re-calibrate when models change — `eval-loop-for-agents` step 4 shares this machinery); and watch for judge-gaming — the generator that learns the judge's lexical tells produces judge-bait, which is why checkable criteria (step 1) hold the perimeter and the judge covers only the interior.

5. **Compose the full stop-condition — done OR converged OR capped, all three armed:**
   ```
   stop when: criteria_met(output)                       # step 1's checks
          or  delta < ε for K consecutive iterations     # step 2's plateau
          or  budget exhausted                           # runaway-cost-guardrails
   each with its own exit label and report
   ```
   The exit label matters downstream: "done" ships, "converged-below-threshold" ships-with-flag or escalates (`human-in-the-loop-checkpoints` — the plateau-under-target case is a genuine judgment call), "capped" is a defect signal feeding step 7. One stop-condition without the other two is a loop with two unhandled endings.

6. **Guard against the two failure modes of self-assessment:** premature completion — the agent declaring done on unverified claims (the criterion must be CHECKED, not asserted: run the tests, count the sources, validate the schema — `agent-loop-design` step 4's verified-observation rule at the exit gate); and oscillating assessment — the critique loop whose judge flip-flops (revision A criticized for X, the X-fix criticized for Y, the Y-fix re-criticized for X — `tool-use-loop-debugging`'s thrash wearing a quality-process costume; the fix: critiques accumulate as a persistent checklist, and a resolved item can't reopen without new evidence).

7. **Audit exits in production and tune the criteria from the exhaust:** exit-mix monitoring (`feedback-loop-instrumentation` step 3 — capped-exits rising means criteria drifted from reality or tasks outgrew the loop); sample "done" outputs for downstream acceptance (done-but-rejected = criteria too loose; the consumer's rejections are the criteria's `regression-test-from-bug` feed); sample "converged-below-threshold" cases for whether the threshold or the loop needs the work. Criteria are a model of the consumer's needs — and models get recalibrated (`metric-definition` step 7's same loop).

## Common pitfalls
- "The agent decides when it's done" with no checkable component: self-assessed completion optimizes for the assessor's optimism — verification-free done-claims are the premature-completion machine (step 6).
- Absolute-threshold-only stopping: the loop that plateaus below the bar and burns the full budget every run, forever — the delta-detector (step 2) exists for exactly this, and it's the most commonly missing piece.
- Aspirational thresholds: 0.95 demanded, 0.90 empirically achievable — every run a budget-exit, diagnosed as "the agent is bad" instead of "the bar was set blind" (step 3's baseline discipline).
- Unanchored judges: "rate this 1–10" drifting with model versions, phrasing, and the weather — rubrics with anchors, calibrated against humans, or the judge is noise with authority (step 4).
- Critique loops without memory: the oscillating judge re-litigating settled points — the persistent checklist (step 6) is two lines of design and saves entire budgets.
- Set-and-forget criteria: the consumer's needs moved, the criteria didn't, and "done" now means "done by 2025's standards" — the production audit (step 7) is the recalibration schedule.

## Example
System: an agent drafting customer-incident summaries from raw timelines (refined via self-critique before human send-out) — v1's stop condition was "the agent judges it ready," producing both 2-iteration undercooked drafts and 19-iteration polishing marathons on different days. Redesign: decomposition (step 1) extracted the checkable majority — all timeline events referenced, impact numbers present and matching the source data, required sections present, length 150–400 words, zero unresolved placeholders (mechanically verified, not asserted); the judged remainder shrank to tone-and-clarity, scored by a rubric'd judge (anchored examples, calibrated at 87% agreement with the support lead's labels on a 60-sample set). Baseline run over 30 historical incidents: quality plateaued at iteration 4–6, judge-score knee at ~4.2/5 with a ceiling of ~4.5 — threshold set at 4.2 (not the aspirational 4.8 someone proposed, which the baseline showed was above the ceiling). Stop-condition composed: checks-pass AND judge ≥ 4.2, OR delta < 0.1 for 2 iterations (exit: converged-flag → human review), OR 10 iterations (exit: capped → defect queue). The oscillation guard earned its place in week one: the judge had been flip-flopping on passive-voice edits — the persistent critique-checklist ended a 3-cycle oscillation pattern visible in the old transcripts. Production month: mean iterations 4.1 (from 9.7), capped-exits 2% (from an effective ~30%), and the downstream audit caught one done-but-rejected class (summaries technically complete but burying the customer-impact line) — fed back as a new checkable criterion (impact-line-in-first-paragraph), the step-7 loop operating as designed.

## Related skills
- `agent-loop-design` — the success exit this skill defines.
- `runaway-cost-guardrails` — the backstop when convergence fails.
- `eval-loop-for-agents` — shared judge-calibration machinery.
- `tool-use-loop-debugging` — the oscillating-critique thrash, general form.
- `slo-definition` — thresholds-from-baselines, the original edition.
