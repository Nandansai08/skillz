---
name: ab-test-analysis
description: >
  Use when designing or analyzing an A/B test — sample size, significance,
  the peeking problem, practical vs statistical significance, segment
  fishing. Triggers: "analyze this A/B test", "is this significant",
  "how long should the experiment run", "sample size for this test",
  "the variant is winning, can we ship?".
---

# A/B Test Analysis

## When to use this skill
- Planning an experiment or reading its results before a ship/no-ship call.
- Auditing someone's "significant win" claim.
- NOT for defining the metric being tested — that's `metric-definition`, and it happens first.

## Prerequisites
- A defined primary metric with guardrails (`metric-definition`).
- Randomization unit chosen (user, session, account) — it must match the metric's denominator unit, or the statistics silently break.

## Workflow

1. **Pre-register the test before starting: one doc, five lines.** Hypothesis ("changing X moves primary metric Y because Z"), primary metric (ONE), guardrails (2–4: retention, latency, revenue — things the change could damage), minimum detectable effect (MDE), and the shipping rule ("ship if primary +≥2% at α=0.05, no guardrail red"). Everything decided after seeing data is tainted by seeing data; this doc is the antidote.

2. **Size the test from the MDE, not the calendar.** The MDE question is business, not stats: "what's the smallest lift that pays for this change?" Then power analysis (any calculator: baseline rate, MDE, α=0.05, power=0.8) → sample per arm → divide by traffic → duration. Two non-negotiables: run **whole weeks** (weekday/weekend populations differ), and if the computed duration is 6 months, the honest answers are "test a bigger change," "use a more sensitive proxy metric," or "don't test — ship-and-watch" — not "run it 2 weeks and squint."

3. **Don't peek — or use methods built for peeking.** Checking daily and stopping on the first significant day inflates false positives massively (a null test "wins" ~30%+ of the time under daily peeking). Options, pick one *in the pre-registration*: fixed horizon (look only at the planned end), group-sequential boundaries (planned interim looks with corrected thresholds), or always-valid/sequential methods (mSPRT — what most experimentation platforms implement). Dashboards everyone can watch + fixed-horizon stats = institutionalized peeking; hide the significance column until the end date if you must.

4. **First analysis step is always the health check, not the result:** **Sample-ratio mismatch** — is the split the configured 50/50? A chi-square on assignment counts; even 50.4/49.6 at scale means broken randomization (bot filtering, redirect losses, crash-before-log in one arm) and **invalidates everything downstream** — diagnose, don't analyze. Then: assignment stability (users switching arms?), pre-experiment metric balance between arms, guardrail scan.

5. **Read the result as effect size + interval, not a verdict:** "+1.8%, 95% CI [+0.3%, +3.3%]" says more than "significant." Check practical vs statistical both ways: significant-but-tiny (huge sample found a real +0.05% — real, but does it pay for the complexity?) and not-significant-but-promising-CI (underpowered, not disproven — the difference matters for what you do next). Statistical significance is a claim about noise, not about worth.

6. **Segments are for hypothesis-generation only.** Slicing results by 20 dimensions yields "significant" segments by arithmetic (20 slices × 5% α = expect one false win). The iron rule: a segment finding from this test gets *confirmed in a new test* targeted at that segment, never shipped from the fishing expedition. Same discipline for secondary metrics: many metrics, same multiplicity problem (or correct formally — Bonferroni/BH — and watch power drop).

7. **Decide by the pre-registered rule and log the outcome** — including losses and flat results, in a searchable experiment log (hypothesis, result, decision). Teams without the log re-run the same failed test every 18 months with new optimism. Flat primary + healthy guardrails + strategic reasons can still justify shipping; that's a legitimate *decision*, just don't launder it as "the test won."

## Common pitfalls
- Stopping early on a good day (peeking, step 3) — the single most common way orgs manufacture false wins.
- Randomizing by session while measuring per-user (unit mismatch): users appear in both arms, variance is miscomputed, the p-value is fiction.
- Ignoring SRM because "it's only half a percent off." SRM is a smoke alarm; the fire is selection bias in one arm.
- Novelty and primacy effects: week-1 lift from "ooh, new button" decaying to zero by week 3 — long enough tests (and looking at the trend of the effect, not just the total) catch it.
- Declaring "no difference" from an underpowered test. Absence of significance ≠ evidence of absence; the CI width tells you what you actually learned.
- Shipping the fished segment ("it won for Android users in Canada!") — step 6's rule exists because that finding is usually noise wearing a narrative.
- Running overlapping experiments on the same surface without an interaction plan or platform-level layering.

## Example
Test: new checkout flow. Pre-registration: primary = checkout completion, MDE 2% relative, guardrails = AOV, support tickets, p95 latency; fixed horizon 3 weeks (power calc: 2 weeks, rounded to whole weeks + buffer). Day 4: PM sees dashboard "+6%, p=0.03", asks to ship — declined per pre-registration (peeking). Health check at end: SRM detected, 50.7/49.3 (p=0.001) — traced to the new flow's extra redirect dropping slow connections *before* the assignment log. Fixed logging, re-ran. Final: +2.4%, CI [+0.9%, +3.9%], guardrails clean except AOV CI wide-but-centered-on-zero — shipped per the rule. The segment slice showed a big mobile-web win; logged as a hypothesis, confirmed in a follow-up test (it held: +5% mobile), which became the next quarter's roadmap item. Experiment log entry #47.

## Related skills
- `metric-definition` — the primary and guardrail metrics come from there.
- `experiment-design` — the general-science version (controls, confounds) this specializes.
- `cohort-retention-analysis` — measuring the long-run effects tests are too short for.
