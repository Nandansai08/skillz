---
name: experiment-design
description: >
  Use when designing an experiment to establish causation — hypotheses,
  controls, confound hunting, randomization, pre-registration mindset —
  for product, engineering, or process changes alike. Triggers: "design an
  experiment for", "how do we test whether X causes Y", "control group",
  "confounds", "pilot program design", "prove this change works".
---

# Experiment Design

## When to use this skill
- A causal claim needs establishing: does this change CAUSE that outcome (process changes, tooling rollouts, product mechanics, performance interventions).
- Reviewing someone's "we tried it and it worked" evidence before it becomes policy.
- NOT the online-A/B-specific machinery (sample-size calculators, SRM, peeking — `ab-test-analysis` owns that specialization); this is the general discipline it specializes from, for settings where clean randomized traffic splits don't exist.

## Prerequisites
- A framed causal question (`research-question-framing` — type: causal) with the outcome variable pinned and measurable (`metric-definition`).
- Honesty about constraints: can you randomize? At what unit? What's the smallest effect worth detecting?

## Workflow

1. **State the hypothesis as a falsifiable prediction with a mechanism:** "X causes Y via M: applying X to [unit], Y will move [direction, roughly how much] within [window]" — the mechanism (WHY would X move Y?) is what separates an experiment from a lottery ticket, and it generates the secondary predictions that let you distinguish "it worked" from "something worked" (if X→Y via faster reviews, review latency should ALSO move; if Y moved and latency didn't, your mechanism is wrong even if your p-value is happy).

2. **Choose the comparison that isolates X — the design's whole job:** concurrent randomized control (the gold standard — randomize at the unit that prevents contamination: teams not individuals if individuals talk); when randomization is impossible, the honest fallbacks in descending strength: staggered/stepped rollout (later adopters as temporary controls), matched comparison (pair treated units with similar untreated ones — and state what "similar" hides), pre/post with a parallel unaffected series (difference-in-differences instinct: your metric AND a sibling metric that X shouldn't touch). Naked pre/post is the weakest design that still gets called an experiment — usable only when the effect is huge, immediate, and nothing else changed (it never is; see step 3).

3. **Hunt confounds BEFORE running, by walking the standard rogues' gallery:** time effects (seasonality, the quarter's deadline crunch, the org reorg that landed mid-experiment) — concurrent controls absorb these, pre/post doesn't; selection effects (volunteers differ from draftees — the pilot team that OPTED IN was already your most motivated); novelty and Hawthorne effects (being observed and being new both move metrics temporarily — run long enough to see the decay); instrumentation changes (the metric's own definition or collection shifting mid-window — freeze it); and contamination (control units adopting the treatment informally because it looked good). For each: either the design neutralizes it, or it's logged as a known limitation — silent confounds are the ones that kill.

4. **Pre-register the analysis — the discipline that keeps you honest with yourself:** before data arrives, write down: primary outcome (ONE), the comparison, the window, the decision rule ("adopt if Y improves ≥Z with the guardrails flat"), guardrail metrics, and the planned segments (if any). This is `ab-test-analysis` step 1 generalized — the point isn't bureaucracy, it's that post-hoc flexibility (choosing the metric/window/segment after seeing data) can manufacture a positive result from noise essentially every time, and the pre-registration is the only known antidote that works on the person running the experiment.

5. **Size and duration by the effect worth detecting:** the smallest effect that would change the decision sets the sample/duration (small units like teams mean small n means only large effects are detectable — SAY so upfront: "this pilot can detect a 30% improvement, not 5%; if we need 5% resolution, this design can't deliver it"); run through complete natural cycles (whole sprints, whole months — partial cycles alias seasonal texture into fake effects); and pre-commit the stop date (early stopping on a good week is the informal peeking problem).

6. **Run with change-control on everything except X:** the experiment window is a no-other-changes zone for the units involved (the tooling migration that lands mid-pilot destroys attribution — coordinate calendars beforehand); log everything that DOES change anyway (the register from step 3 stays open); and blind what can be blinded (the grader of a subjective outcome shouldn't know which units got X — subjective metrics + unblinded graders = the grader's hopes, measured).

7. **Analyze against the pre-registration, report mechanism and limits:** primary outcome vs decision rule first; the mechanism's secondary predictions checked (step 1's payoff); effect with its uncertainty, not just direction (`ab-test-analysis` step 5's interval discipline); confound register published with the result ("selection: pilot self-selected; mitigated by matched comparison, residual risk stated"); and the explicit external-validity sentence — what contexts this finding does and doesn't license ("held for backend teams on the monorepo; frontend's review dynamics differ — separate pilot before extending"). Then log it win-or-lose (`ab-test-analysis` step 7's same registry) — negative results are paid-for knowledge, and unlogged ones get expensively re-purchased.

## Common pitfalls
- The volunteer pilot generalized: the opt-in team improved 25%, policy rolled to everyone, nothing moved — selection was the treatment (step 3's most common rogue in org settings).
- Pre/post across a boundary: measured before-vacation vs after-vacation, credited the tool. Time is the eternal confound; concurrent comparison or diff-in-diff (step 2).
- Mechanism-free wins: Y moved, adopt X! — with no mechanism check, you've learned a correlation with ceremony; when the context shifts, the "effect" evaporates and nobody knows why (step 1's secondary predictions are the cheap insurance).
- Outcome shopping: seven metrics measured, the one that moved reported — pre-registration (step 4) or it's noise mining with a lab coat.
- Underpowered theater: n=3 teams, 4 weeks, "no significant difference" reported as "X doesn't work" — the design could only have detected miracles; absence of evidence at this power is just absence (step 5's detectability confession).
- The eternal pilot: no pre-committed decision rule or stop date, so it runs until the sponsor's patience or the advocate's hope wins — which is a politics outcome wearing an experiment's badge.

## Example
Causal question (framed upstream): "does AI-assisted review reduce escaped defects, via catching more issues pre-merge?" Design: 12 backend teams, randomized 6/6 at the TEAM unit (individuals share channels — contamination), 2 complete sprint-cycles, concurrent control. Pre-registered: primary = escaped-defect rate (production bugs traced to the window's merges); mechanism check = pre-merge findings per PR should RISE in treatment; guardrails = review latency, developer-reported friction; decision rule = adopt org-wide if defects −20%+ with guardrails flat; segments: none. Confound register: novelty (mitigated: second sprint weighted), instrumentation (defect-tracing rubric frozen and blinded — the grader didn't know team assignments), one mid-window platform migration (rescheduled after a calendar check — step 6 earning its keep). Result: defects −24% [CI −8% to −38%], mechanism confirmed (findings/PR +0.9), latency flat, friction slightly up (noted). The mechanism check did real work: one treatment team showed NO defect change AND no findings change — investigation revealed they'd disabled the tool in week one (non-compliance, not treatment failure — reported as such rather than diluting the effect). Adopted per the rule; the external-validity sentence routed frontend to its own smaller pilot, which — with different review dynamics — showed nothing, exactly as the sentence had licensed suspecting.

## Related skills
- `ab-test-analysis` — the high-volume online specialization.
- `research-question-framing` — the causal question this design answers.
- `metric-definition` — the outcome variables' rigor.
- `cohort-retention-analysis` — observational cousin; its correlations feed this skill's hypotheses.
