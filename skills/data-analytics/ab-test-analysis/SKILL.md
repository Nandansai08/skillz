---
name: ab-test-analysis
description: >
  Use when designing or analyzing an A/B test — sample size, significance,
  the peeking problem, practical vs statistical significance, segment
  fishing. Triggers: "analyze this A/B test", "is this significant", "how
  long should the experiment run", "sample size for this test", "the
  variant is winning, can we ship?". NOT for defining the metric being
  tested (see metric-definition, which comes first) and NOT for offline/
  org experiments without traffic splits (see experiment-design).
---

# A/B Test Analysis

## Overview

Online experiments manufacture false wins through exactly three doors — peeking, unit mismatch, and segment fishing — and pre-registration closes all three. The health check (SRM first) comes before any result, because a broken split invalidates everything downstream.

## When to Use

- Planning an experiment or reading its results before a ship/no-ship call.
- Auditing someone's "significant win" claim.

**When NOT to use:**
- Defining the metric — `metric-definition`, first.
- Causal questions without randomizable traffic — `experiment-design`.

## Prerequisites

- A defined primary metric with guardrails (`metric-definition`).
- Randomization unit chosen (user, session, account) — it must match the metric's denominator unit, or the statistics silently break.

## The Workflow

1. **Pre-register the test before starting: one doc, five lines.** Hypothesis ("changing X moves primary metric Y because Z"), primary metric (ONE), guardrails (2–4: retention, latency, revenue), minimum detectable effect (MDE), and the shipping rule ("ship if primary +≥2% at α=0.05, no guardrail red"). Everything decided after seeing data is tainted by seeing data; this doc is the antidote.

2. **Size the test from the MDE, not the calendar.** The MDE question is business, not stats: "what's the smallest lift that pays for this change?" Then power analysis (baseline rate, MDE, α=0.05, power=0.8) → sample per arm → duration. Two non-negotiables: run **whole weeks** (weekday/weekend populations differ), and if the computed duration is 6 months, the honest answers are "test a bigger change," "use a more sensitive proxy metric," or "ship-and-watch" — not "run it 2 weeks and squint."

3. **Don't peek — or use methods built for peeking.** Checking daily and stopping on the first significant day inflates false positives massively (a null test "wins" ~30%+ under daily peeking). Options, picked *in the pre-registration*: fixed horizon, group-sequential boundaries, or always-valid methods (mSPRT — what most platforms implement). Dashboards everyone watches + fixed-horizon stats = institutionalized peeking; hide the significance column until the end date if you must.

4. **First analysis step is always the health check, not the result:** **Sample-ratio mismatch** — is the split the configured 50/50? Chi-square on assignment counts; even 50.4/49.6 at scale means broken randomization (bot filtering, redirect losses, crash-before-log in one arm) and **invalidates everything downstream** — diagnose, don't analyze. Then: assignment stability, pre-experiment metric balance, guardrail scan.

5. **Read the result as effect size + interval, not a verdict:** "+1.8%, 95% CI [+0.3%, +3.3%]" says more than "significant." Check practical vs statistical both ways: significant-but-tiny (real, but does it pay for the complexity?) and not-significant-but-promising-CI (underpowered, not disproven — absence of significance ≠ evidence of absence; the CI width says what you actually learned).

6. **Segments are for hypothesis-generation only.** Twenty slices × 5% α = expect one false "win" by arithmetic. The iron rule: a segment finding gets *confirmed in a new targeted test*, never shipped from the fishing expedition. Same for secondary metrics — same multiplicity problem (or correct formally and watch power drop).

7. **Decide by the pre-registered rule and log the outcome** — including losses and flats, in a searchable experiment log (hypothesis, result, decision). Teams without the log re-run the same failed test every 18 months with new optimism. Flat primary + healthy guardrails + strategic reasons can still justify shipping; that's a legitimate *decision* — just don't launder it as "the test won." Watch for novelty effects: week-1 lift decaying by week 3 is why trends of the effect get read, not just totals.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "It's at p=0.03 on day 4 — ship before we lose the window" | Daily peeking manufactures wins from null effects at ~30% rates. The day-4 significance is exactly what the pre-registered horizon exists to ignore. |
| "SRM is only half a percent off — close enough" | SRM is a smoke alarm: the imbalance means one arm is silently losing a nonrandom slice (crashes, bots, redirects), and every downstream number inherits the bias. Diagnose, never analyze through it. |
| "Not significant means no effect — kill the feature" | An underpowered test's flat result is 'we didn't learn,' not 'it doesn't work.' The CI width distinguishes the two; the verdict-word hides it. |
| "It won for Android users in Canada — roll out to that segment!" | Twenty slices guarantee a fake winner by arithmetic. Fished segments are hypotheses for the NEXT test; shipping them is noise-worship with a rollout plan. |
| "We'll define success criteria once we see how it performs" | Post-hoc criteria are chosen to bless whatever happened — the test becomes a ceremony. Five lines before launch (step 1) is the entire integrity mechanism. |
| "Randomize sessions, measure users — same thing roughly" | Users land in both arms, variance is miscomputed, and the p-value is fiction. Unit match isn't pedantry; it's what makes the statistics statistics. |

## Red Flags

- A "winner" declared before the pre-registered end date.
- No SRM check anywhere in the analysis.
- Results quoted as verdicts with no effect size or interval.
- The primary metric changed between launch and readout.
- Segment wins shipping without confirmation tests.
- No experiment log; the same idea tested twice in two years, unknowingly.

## Verification

- [ ] Pre-registration doc exists, dated before launch: hypothesis, one primary, guardrails, MDE, ship rule — link.
- [ ] Power calc shown; duration in whole weeks — numbers attached.
- [ ] SRM chi-square passed (or the test invalidated and rerun) — assignment counts shown.
- [ ] Result reported as effect + CI; ship decision matches the pre-registered rule — readout linked.
- [ ] Any segment/secondary findings marked as hypotheses, not shipped — noted.
- [ ] Experiment logged (win, loss, or flat) in the searchable registry — entry linked.

## Example

Test: new checkout flow. Pre-registration: primary = checkout completion, MDE 2% relative, guardrails = AOV, support tickets, p95 latency; fixed horizon 3 weeks. Day 4: PM sees dashboard "+6%, p=0.03", asks to ship — declined per pre-registration (peeking). Health check at end: SRM detected, 50.7/49.3 (p=0.001) — traced to the new flow's extra redirect dropping slow connections *before* the assignment log. Fixed logging, re-ran. Final: +2.4%, CI [+0.9%, +3.9%], guardrails clean — shipped per the rule. The segment slice showed a big mobile-web win; logged as a hypothesis, confirmed in a follow-up test (it held: +5% mobile), which became the next quarter's roadmap item. Experiment log entry #47.

## Related skills

- `metric-definition` — the primary and guardrail metrics come from there.
- `experiment-design` — the general-science version this specializes.
- `cohort-retention-analysis` — measuring the long-run effects tests are too short for.
