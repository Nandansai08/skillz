---
name: agent-loop-design
description: >
  Use when structuring an agentic system's core loop — plan→act→observe→
  reflect phases, exit conditions, state carried between iterations —
  before writing the agent code. Triggers: "design an agent loop",
  "structure this agent", "my agent wanders off task", "agentic workflow
  architecture", "plan-act-observe", "when should the agent stop".
---

# Agent Loop Design

## When to use this skill
- Building an LLM agent (or any autonomous iterative system) and deciding its loop structure.
- An existing agent wanders, stalls, or overshoots — symptoms of missing loop discipline, diagnosed here, debugged in `tool-use-loop-debugging`.
- NOT for single-shot LLM calls or fixed pipelines — if the steps are known in advance, you want a workflow, not a loop (and that realization is this skill's cheapest win).

## Prerequisites
- The task's success criteria, statable in advance — an agent loop without a checkable "done" is `convergence-criteria-design`'s problem; solve it first or concurrently.
- Honest answer to "does this need an agent at all?": loops earn their cost when the path is unknowable upfront; a known 5-step process wrapped in an agent is expensive nondeterminism.

## Workflow

1. **Structure each iteration as plan → act → observe → reflect, with real content in each phase:**
   - **Plan:** the agent states what it's about to do and why, against the goal ("next: read the failing test, because the error names it") — plans forced into words are plans that can be checked, logged, and debugged.
   - **Act:** ONE bounded action (a tool call, an edit) — multi-action bundles blur attribution when something breaks.
   - **Observe:** the action's actual result captured and summarized — not assumed ("the edit applied" vs verifying it did; unverified observation is where agent state diverges from world state, the root of most agent failure).
   - **Reflect:** progress assessed against the goal — closer, stuck, or off-track — and the verdict feeds the next plan. Skipping reflect turns the loop into a stimulus-response chain that can't notice its own drift.

2. **Define the state that crosses iterations, explicitly and minimally:** the goal (immutable, restated where the model can always see it), the plan/progress ledger (what's done, what remains), key observations worth keeping, and the attempt history for the current obstacle (what's been tried — the input `retry-and-backoff-strategy` and thrash-detection need). Everything else is candidate for dropping — unbounded state accumulation is `context-window-loop-management`'s territory; design the state schema NOW so that skill has something to manage.

3. **Write the exit conditions before the loop body — all four kinds:**
   - **Success:** the checkable done-condition (tests pass, the answer validates — `convergence-criteria-design` for the hard cases).
   - **Failure:** the agent concludes the task is impossible/blocked, with the reason surfaced (an agent that can't give up converts impossible tasks into infinite loops).
   - **Budget:** iteration/token/time caps (`runaway-cost-guardrails` owns the numbers) — the backstop that holds when the other exits fail.
   - **Escalation:** conditions that hand control to a human (`human-in-the-loop-checkpoints`) — destructive actions, confidence collapse, repeated failure.
   A loop with only the success exit has three unhandled failure modes by construction.

4. **Make every action's outcome observable to the loop:** tool results returned in full-enough fidelity to act on (the truncated error message that hides the actual cause), side effects verified (file written? test actually ran?), and failures as first-class observations, not exceptions that skip the reflect phase — the agent that never sees its own errors repeats them (`feedback-loop-instrumentation` is the logging half of this; here it's the design half: the loop's information diet determines its ceiling).

5. **Choose the structure's rigidity deliberately** (`state-machine-vs-freeform-loop` owns the full decision): freeform plan-act loops for open exploration; phased structure (research → plan → execute → verify, each phase its own loop with its own exit) when the task has natural stages — phase boundaries are where drift gets caught and budgets get checkpointed. The common sweet spot: rigid outer phases, freeform inner iterations.

6. **Design the failure-handling ladder inside the loop:** action fails → retry policy for the transient class (`retry-and-backoff-strategy`); same approach fails twice → the reflect phase must generate a DIFFERENT approach, not a third identical attempt (the two-state thrash is `tool-use-loop-debugging`'s most common patient — design the "have I tried this before?" check against step 2's attempt history NOW); no approaches left → the failure or escalation exit, with the attempt history as the handoff report.

7. **Dry-run the design against three scenarios on paper before coding:** the happy path (does state accumulate sensibly? do phases hand off cleanly?), the stuck path (obstacle that survives all approaches — does it exit with a useful report, or spin?), and the drift path (a plausible mid-task distraction — does anything pull it back to the goal? the goal-restatement in step 2 and the reflect-against-goal in step 1 are the anti-drift machinery; verify they'd fire). Ten minutes of paper-running catches the majority of loop-design defects at zero token cost.

## Common pitfalls
- The reflect-free loop: act-observe-act-observe, no progress assessment — the agent that works hard, drifts freely, and can't tell (step 1's fourth phase is the one most often skipped and most expensive to skip).
- Success-only exits: everything else becomes an infinite loop with a token bill (`runaway-cost-guardrails` exists because this pitfall is the norm, not the exception).
- Assumed observations: the agent proceeds on what it INTENDED, not what happened — state diverges from world, and every subsequent iteration compounds the fiction (step 4's verification).
- Kitchen-sink state: everything ever observed carried forever — context exhaustion wearing a thoroughness costume (step 2's minimalism; `context-window-loop-management` for the ongoing diet).
- Agent-shaped workflows: a deterministic 5-step process given agency it doesn't need — slower, costlier, and occasionally creative in the bad way. The prerequisite question, asked honestly.
- Designing only the happy path: the stuck and drift scenarios (step 7) are where agents actually live; a loop untested against them on paper gets tested against them in production, at market rates.

## Example
Task: an agent that fixes failing CI builds. First design (rejected on the paper dry-run): freeform loop, "fix the build" goal, success-only exit — the stuck-path scenario showed it re-attempting the same dependency pin forever. Shipped design: phased outer structure (diagnose → propose → apply → verify), each phase exit-conditioned; state schema = goal + failure summary + attempt ledger + current hypothesis (nothing else carried — raw logs summarized at diagnose-exit per `context-window-loop-management`); exits = success (CI green), failure ("root cause outside repo" — with evidence), budget (12 iterations / 15 min), escalation (any change touching deploy configs → human per `human-in-the-loop-checkpoints`). The reflect phase's thrash check: any hypothesis attempted twice forces a new hypothesis or the failure exit. Production month one: 61% of failures fixed autonomously; the design's proof was the OTHER 39% — every non-fix exited with a diagnosis report (the failure exit doing its job), zero runaway loops, two escalations both legitimate. The attempt-ledger requirement caught the design's one production bug: a flaky-test class where the agent's "different approach" generator produced semantically-identical retries — patched by adding the attempted-diff hash to the ledger (`tool-use-loop-debugging`'s signature move, pre-wired by this design).

## Related skills
- `convergence-criteria-design` — the "done" definition this loop checks against.
- `runaway-cost-guardrails` — the budget exits' numbers.
- `state-machine-vs-freeform-loop` — the rigidity decision in step 5.
- `tool-use-loop-debugging` — when the built loop misbehaves anyway.
- `context-window-loop-management` — the state diet across iterations.
