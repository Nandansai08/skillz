---
name: experiment-design
description: >
  Use when designing an experiment to establish causation — hypotheses,
  controls, confound hunting, randomization, pre-registration mindset —
  for product, engineering, or process changes alike. Triggers: "design
  an experiment for", "how do we test whether X causes Y", "control
  group", "confounds", "pilot program design", "prove this change works".
  NOT the online-A/B machinery (see ab-test-analysis) — this is the
  general discipline for settings without clean traffic splits.
---

# Experiment Design

## Overview

An experiment is a comparison that isolates X plus a mechanism that predicts more than the headline metric — without both, "we tried it and it worked" is a lottery ticket with a methods section. The confound gallery is walked BEFORE running, because silent confounds are the ones that kill.

## When to Use

- A causal claim needs establishing: process changes, tooling rollouts, product mechanics.
- Reviewing someone's "we piloted it and it worked" evidence before it becomes policy.

**When NOT to use:**
- High-volume online splits — `ab-test-analysis` owns the SRM/peeking/sizing machinery.

## Prerequisites

- A framed causal question (`research-question-framing`) with the outcome pinned and measurable (`metric-definition`).
- Honesty about constraints: can you randomize? At what unit? Smallest effect worth detecting?

## The Workflow

1. **State the hypothesis as a falsifiable prediction with a mechanism:** "X causes Y via M: applying X to [unit], Y moves [direction, roughly how much] within [window]." The mechanism generates secondary predictions that distinguish "it worked" from "something worked" — if X→Y via faster reviews, review latency should ALSO move; Y moving without it means your mechanism is wrong even if your p-value is happy.

2. **Choose the comparison that isolates X — the design's whole job:** concurrent randomized control (gold standard — randomize at the unit that prevents contamination: teams, not individuals who talk); when randomization is impossible, the honest fallbacks in descending strength: staggered rollout (later adopters as temporary controls), matched comparison (state what "similar" hides), pre/post with a parallel unaffected series (difference-in-differences instinct). Naked pre/post is the weakest design still called an experiment — usable only when the effect is huge, immediate, and nothing else changed (it never is).

3. **Hunt confounds BEFORE running, walking the rogues' gallery:** time effects (seasonality, the deadline crunch, the reorg — concurrent controls absorb these; pre/post doesn't); selection effects (volunteers differ from draftees — the opt-in pilot team was already your most motivated); novelty/Hawthorne (run long enough to see the decay); instrumentation changes (freeze the metric's definition); contamination (control units adopting informally). Per confound: the design neutralizes it, or it's logged as a known limitation — silent confounds are the killers.

4. **Pre-register the analysis:** before data, write: primary outcome (ONE), comparison, window, decision rule, guardrails, planned segments. Post-hoc flexibility manufactures positives from noise essentially every time; pre-registration is the only antidote that works on the person running the experiment (`ab-test-analysis` step 1, generalized).

5. **Size and duration by the effect worth detecting:** small units (teams) mean small n means only large effects detectable — SAY so upfront ("this pilot detects a 30% improvement, not 5%"); run complete natural cycles (whole sprints/months — partial cycles alias seasonality into fake effects); pre-commit the stop date (early stopping on a good week is informal peeking).

6. **Run with change-control on everything except X:** the window is a no-other-changes zone for involved units (coordinate calendars beforehand — the mid-pilot migration destroys attribution); log what changes anyway; blind what can be blinded (a subjective outcome's grader shouldn't know which units got X — unblinded graders measure their own hopes).

7. **Analyze against the pre-registration; report mechanism and limits:** primary vs decision rule first; the mechanism's secondary predictions checked (step 1's payoff); effect WITH uncertainty; the confound register published with the result; and the explicit external-validity sentence — what contexts this licenses ("held for backend teams on the monorepo; frontend differs — separate pilot"). Log it win-or-lose — negative results are paid-for knowledge, and unlogged ones get expensively re-purchased.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "The pilot team improved 25% — roll it out" | The pilot team OPTED IN — selection was the treatment. The org-wide rollout of a volunteer effect moves nothing, a quarter late (the gallery's most common rogue). |
| "Before vs after shows the improvement clearly" | Time is the eternal confound: the 'after' also contains the new quarter, the training decay, the seasonal lull. Concurrent comparison or diff-in-diff; naked pre/post is a chart, not evidence. |
| "Y moved — who cares about the mechanism?" | Without the mechanism check, you've learned a correlation with ceremony; when context shifts, the 'effect' evaporates and nobody knows why. The secondary predictions are the cheap insurance. |
| "We measured seven outcomes to be thorough" | And will report the one that moved — noise mining with a lab coat. One pre-registered primary; the other six are exploratory, labeled. |
| "No significant difference at n=3 teams — X doesn't work" | The design could only detect miracles; absence of evidence at that power is just absence. The detectability confession (step 5) belongs in the proposal, not the postmortem. |
| "Extend the pilot — the trend looks promising" | The eternal pilot with no stop date runs until the sponsor's patience or the advocate's hope wins: a politics outcome wearing an experiment's badge. |

## Red Flags

- Opt-in pilots generalized to mandates.
- Pre/post across a quarter boundary, a reorg, or a season.
- No mechanism prediction anywhere.
- Metrics chosen after data arrived.
- Subjective outcomes graded by unblinded advocates.
- No stop date; no decision rule; "we'll see how it goes."
- Negative results vanishing unlogged.

## Verification

- [ ] Hypothesis with mechanism + secondary predictions — written, dated pre-run.
- [ ] Comparison design named with its strength class; randomization unit justified against contamination.
- [ ] Confound register: each rogue neutralized-by-design or logged as limitation.
- [ ] Pre-registration doc: one primary, rule, window, guardrails, segments — dated before data.
- [ ] Detectable-effect size stated upfront; complete cycles; stop date pre-committed.
- [ ] Report: primary vs rule, mechanism checks, uncertainty, confound register, external-validity sentence — logged win-or-lose.

## Example

Causal question (framed upstream): "does AI-assisted review reduce escaped defects, via catching more issues pre-merge?" Design: 12 backend teams, randomized 6/6 at the TEAM unit (individuals share channels — contamination), 2 complete sprint-cycles, concurrent control. Pre-registered: primary = escaped-defect rate; mechanism check = pre-merge findings per PR should RISE in treatment; guardrails = review latency, developer friction; decision rule = adopt if defects −20%+ with guardrails flat. Confound register: novelty (second sprint weighted), instrumentation (defect-tracing rubric frozen, grader blinded to assignments), one mid-window platform migration (rescheduled after a calendar check). Result: defects −24% [CI −8% to −38%], mechanism confirmed (findings/PR +0.9), latency flat. The mechanism check did real work: one treatment team showed NO change in either metric — investigation revealed they'd disabled the tool in week one (non-compliance, not treatment failure — reported as such). Adopted per the rule; the external-validity sentence routed frontend to its own smaller pilot, which showed nothing — exactly as the sentence had licensed suspecting.

## Related skills

- `ab-test-analysis` — the high-volume online specialization.
- `research-question-framing` — the causal question this design answers.
- `metric-definition` — the outcome variables' rigor.
- `cohort-retention-analysis` — observational cousin; its correlations feed this skill's hypotheses.
