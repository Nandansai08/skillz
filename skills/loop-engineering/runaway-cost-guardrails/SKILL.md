---
name: runaway-cost-guardrails
description: >
  Use when putting hard limits on agent loops — iteration caps, token and
  spend budgets, time limits, and the layered enforcement that stops a
  runaway before the invoice does. Triggers: "cap the agent's spend",
  "token budget", "agent ran all night", "iteration limit", "cost blew up",
  "kill switch for the loop". NOT for tuning convergence so loops stop
  naturally (see convergence-criteria-design) — these are the backstops
  for when that fails.
---

# Runaway Cost Guardrails

## Overview

Every loop eventually meets a task that defeats its stop conditions — the guardrails decide whether that costs twelve dollars or twelve thousand. Budgets are not the stopping strategy; they're the enforcement layer that holds when the stopping strategy fails.

## When to Use

- Deploying any agent loop beyond a supervised notebook — guardrails ship WITH the loop, not after its first incident.
- A runaway just happened (the overnight bill, the 400-iteration spiral) and the postmortem needs the fix.
- Setting fleet-level spend policy across many agents/users.

**When NOT to use:**
- Making loops stop at good-enough — that's `convergence-criteria-design`; a loop relying on budget caps as its NORMAL exit is misconfigured upstream.
- Per-call API retry tuning — `retry-and-backoff-strategy` (its retries draw from these budgets, but the mechanics live there).

## Prerequisites

- Cost/iteration distributions from real or pilot runs (`feedback-loop-instrumentation` step 3's p50/p95) — caps set without distributions are either theater (never fire) or throttles (fire on normal work).

## The Workflow

1. **Cap all four axes — they fail independently:**
   - **Iterations:** the spiral catcher (cheap loops can spin thousands of times inside any token budget).
   - **Tokens/spend:** the direct money bound, per-run.
   - **Wall-clock time:** catches the hung tool call and the slow-bleed loop that's within both other caps.
   - **Action-class counts:** caps on expensive/risky specific actions (external API calls with per-call cost, message sends, VM spawns) — the axis teams forget, and the one that turns "one run" into "4,000 emails."
   A single-axis guardrail is a three-door barn.

2. **Set numbers from the distribution, not from fear or hope:** cap ≈ p95-of-legitimate-runs × 2–3 margin, per task class (a triage task and a migration task deserve different budgets — one cap for both either strangles the big task or unleashes the small one). Warning: caps set at p50 convert half your normal workload into "runaways"; the resulting alert fatigue gets the guardrails deleted within a quarter, which is worse than never having them (`alerting-design` step 7's actionability discipline applies to budget alarms verbatim).

3. **Layer the enforcement — inner layers advise, outer layers compel:**

   ```
   Layer 1: in-loop budget awareness (agent sees remaining budget,
            plans against it, wraps up gracefully)      [cooperative]
   Layer 2: harness hard stop (the loop runner kills the run
            at cap — agent cooperation not required)     [mechanical]
   Layer 3: platform limits (API key spend caps, rate limits,
            org-level quotas — survives harness bugs)    [external]
   Layer 4: billing alerts + kill switch (the human layer,
            for when 1–3 were all misconfigured)         [last resort]
   ```

   Layer 1 alone is a request, not a limit — the runaway case is precisely the agent whose judgment already failed. Layers 2 and 3 are the actual guardrails; layer 3 especially, because it survives the harness's own bugs (the crashed-loop-that-retries-itself incident lives below layer 2's reach).

4. **Design the budget-exit as a first-class outcome, not a crash:** on cap: persist state and partial results, emit the exit-labeled trace (`feedback-loop-instrumentation` step 1's exit reasons), produce the handoff report (what was attempted, where it stood, why it likely didn't converge — the attempt ledger as receipt), and route by task type: park-for-human, queue-for-bigger-budget-retry (ONCE, not recursively — auto-retry-with-more-budget is a runaway with extra steps), or fail-visibly. A budget kill that loses all partial work converts every cap event into a total loss, which pressures everyone to raise caps — the failure spiral of guardrail design.

5. **Give the agent graceful-degradation awareness near the line:** at ~80% budget, the loop's reflect phase gets the signal ("2 iterations remain") — competent behavior is wrapping up: committing partial results, writing the summary, choosing the checkable subset of remaining work. This is `agent-loop-design` step 3's budget exit made cooperative — the difference between a run that ends mid-keystroke and one that lands.

6. **Aggregate the budgets above the run level:** per-user/per-tenant daily caps (one user's retry-happy script shouldn't spend the fleet's day), per-agent-type fleet budgets, and global spend-rate alarms (spend-per-hour deviation from baseline — the earliest fleet-level runaway signal, since individual runs can all be under-cap while a bug launches 50× the normal run count: the run-LAUNCHING loop is a runaway the per-run caps can't see).

7. **Test the guardrails like the safety equipment they are:** deliberately run an unbounded task (a goal that can't converge) in staging and verify each layer fires in order — layer 1's graceful wrap attempt, layer 2's kill at cap, state persisted, report emitted; verify layer 3 by letting a harness-bypass simulation hit the platform cap. Review quarterly: budget-exit rate per task class (rising = tasks outgrew caps or quality regressed — either way, news: `eval-loop-for-agents` correlates), and cap-vs-p95 drift as the workload evolves. An untested kill switch is a decoration with a comforting name (`deployment-rollback-plan` step 7's rule, agent edition).

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "The agent is well-designed; it won't run away" | The guardrail exists for the day the design meets the input it didn't anticipate. Well-designed systems still ship circuit breakers — the quality of the loop and the need for the backstop are independent. |
| "We told it to stop after 10 iterations in the prompt" | A prompt is layer 1 — cooperative, and specifically unreliable in the failure mode it's guarding (the agent whose judgment has already degraded). Layer 2's mechanical stop is the actual limit. |
| "Caps will kill legitimate long tasks" | Caps set at p50 will. Caps at p95×2–3, per task class, fire on the pathological tail only — and the budget-exit's retry-once lane (step 4) catches the rare legitimate giant. |
| "We have billing alerts, that's our guardrail" | Billing alerts are layer 4 — they fire in hours-to-days granularity, after the money's gone. They're the smoke detector for when the sprinklers (layers 2–3) failed, not the sprinklers. |
| "It's cheap per run — capping isn't worth the engineering" | Cheap per run × unbounded runs is the exact shape of the incident (step 6's launching-loop). The per-run cap is an afternoon; the aggregate caps are a config file. |
| "We'll add limits after we see how it behaves in prod" | Prod IS where the tail input lives. Observing without caps means the first observation of the tail is the invoice — pilot distributions (prerequisites) exist so caps precede exposure. |

## Red Flags

- Any deployed loop whose only stop conditions are success and the agent's own judgment.
- Budget caps that have never fired in months of operation (either miracles, or theater — verify which via step 7's drill).
- Budget-exit rate quietly climbing while everyone celebrates "the agent handles harder tasks now."
- Cap events that lose all partial work — and the correlated pressure to "just raise the cap."
- Per-run caps present, per-user/fleet aggregates absent.
- The kill-switch drill never run; nobody can say which layer would catch a harness crash-loop.
- Auto-retry-on-budget-exit with escalating budgets and no retry cap.

## Verification

- [ ] All four axes capped (iterations, spend, time, action-class) — config linked.
- [ ] Cap values derived from measured distributions, derivation written next to the config (p95 source linked).
- [ ] Layers 2 AND 3 verified live: staging drill trace showing the harness kill, and the platform-level cap tested independently — both artifacts linked.
- [ ] Budget-exit produces persisted state + handoff report — sample exit trace attached.
- [ ] Aggregate caps (per-user/tenant, fleet spend-rate alarm) configured — links.
- [ ] Exit-reason mix on the dashboard; budget-exit rate reviewed at a named cadence with an owner.

## Example

Incident: a repo-maintenance agent hit a git edge case (detached HEAD its tools couldn't diagnose), spun overnight — 2,100 iterations, $3,400, discovered by the morning's billing email (layer 4 doing its lonely job; layers 1–3 didn't exist). Rebuild per the ladder: distributions pulled from two weeks of traces (p95 = 14 iterations, $1.80 for the maintenance class) → caps at 40 iterations / $6 / 20 minutes / 3 external-API calls per run; layer 2 in the harness with state-persist + handoff report on kill; layer 3 as per-key daily spend caps at the provider ($200/day for the whole agent class — the cap that would have turned $3,400 into $200 even if everything else failed); fleet spend-rate alarm at 3× baseline hourly. The graceful layer earned its keep unexpectedly: budget-visibility in the reflect phase cut MEAN cost 12% — runs started self-selecting smaller verification steps when low, an unplanned efficiency. The staging drill (unfixable-goal task) verified the cascade: wrap-up attempt at iteration 32, hard kill at 40, report emitted, state resumable. Quarter two's review caught the drift case: budget-exits on the "dependency update" class rose 2%→9% — not runaways, but a new monorepo whose legitimate runs needed more; that class's cap re-derived from its new p95, everyone else's unchanged. The original detached-HEAD case is pinned in the eval suite, where it now exits at iteration 6 with a correct "cannot resolve, needs human" report — the failure that built the guardrails, retired to teaching duty.

## Related skills

- `convergence-criteria-design` — the stopping strategy these rails back up.
- `agent-loop-design` — the budget exit as one of the four mandatory exits.
- `feedback-loop-instrumentation` — the distributions and exit-mix telemetry.
- `retry-and-backoff-strategy` — retries as budget consumers, and the same layered-defense instinct.
- `human-in-the-loop-checkpoints` — the sibling safety system for risk (this one is for spend).
