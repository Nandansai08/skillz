---
name: state-machine-vs-freeform-loop
description: >
  Use when deciding how much structure to impose on an agent — explicit
  states with defined transitions versus free reasoning each iteration,
  and the hybrid points between. Triggers: "should this be a state
  machine", "how much structure for the agent", "constrain the agent or
  let it reason", "workflow vs agent", "phases or freeform". NOT for
  designing the chosen loop's internals (see agent-loop-design) or for
  debugging a structure that already exists (see tool-use-loop-debugging).
---

# State Machine vs Freeform Loop

## Overview

The most consequential agent-architecture decision is how much freedom each iteration gets: explicit states buy predictability, debuggability, and enforceable gates; freeform reasoning buys adaptability to paths you didn't foresee. Choosing wrong in either direction is expensive — this skill is the decision procedure.

## When to Use

- Architecting a new agent and choosing its control structure.
- An existing freeform agent is unpredictable in ways that keep costing (drift, skipped safety steps) — candidate for adding structure.
- An existing rigid pipeline keeps failing on tasks that don't fit its stages — candidate for loosening.

**When NOT to use:**
- The internals of whichever structure you pick — `agent-loop-design` owns phase content, state schemas, exits.
- Pure deterministic pipelines with no LLM decisions — that's just a workflow engine; no agent decision exists.

## Prerequisites

- A sample of real tasks the system will handle (10+, spanning easy to hard) — the decision is driven by the task distribution's shape, and guessing the shape is how both failure modes get built.

## The Workflow

1. **Classify the task distribution's path-predictability.** For each sample task, sketch the solution path. Then ask: do the paths share a stable skeleton?

   ```
   Do 80%+ of tasks follow the same step sequence?
        |                              |
       yes                             no
        |                              v
        v                     Are there ANY invariant
   State machine /            checkpoints all paths share
   staged pipeline            (validate, gate, verify)?
   (freeform only inside           |            |
    steps that need it)           yes           no
                                   |            |
                                   v            v
                          Hybrid: rigid     Freeform loop
                          outer phases,     (plan-act-observe-
                          freeform inside   reflect + budgets)
   ```

   Most production systems land in the hybrid cell — and most architecture mistakes come from not noticing that.

2. **Weigh the structural forces beyond path-shape.** Toward MORE structure: safety-critical actions needing enforced gates (`human-in-the-loop-checkpoints` — a checkpoint inside a freeform loop is a suggestion; as a state transition it's a wall), compliance/audit needs (states are legible to auditors; reasoning traces less so), many similar runs at scale (predictable cost), weaker/cheaper models (structure substitutes for judgment), multi-agent handoffs (states are the natural contract). Toward LESS: genuinely novel tasks per run, exploration/research shapes, strong models on hard problems where your predefined stages would be the ceiling, low volume where building the machine costs more than supervising the agent.

3. **If state machine: make states real, not decorative.** Each state gets: entry condition, allowed actions (enforced by tool availability, not prompt hope — the state that "shouldn't" call deploy tools simply doesn't have them), exit criteria (checkable — `convergence-criteria-design` per state), and defined transitions including failure transitions (state X fails → retry state / fallback state / escalate — the missing failure transition is where state machines silently become freeform). Keep the count honest: 3–7 states; a 20-state machine is a flowchart cosplaying as robustness, and every state is surface area to maintain.

4. **If freeform: buy back predictability with invariants.** The freeform loop still gets: immutable goal restatement, budget rails (`runaway-cost-guardrails`), attempt ledger and repetition detection (`tool-use-loop-debugging` step 6), and non-negotiable verification before "done" (`agent-loop-design`). Freeform means the PATH is free — never that the guardrails are.

5. **For the hybrid (the usual answer): draw the boundary at trust transitions.** Rigid where mistakes are expensive or gates are mandatory (input validation, plan approval, final verification, anything irreversible); freeform where the work is genuinely exploratory (diagnosis, research, creative synthesis). The pattern that recurs: rigid outer skeleton `intake → explore(freeform) → plan → approve(gate) → execute(freeform within plan) → verify`, with each freeform pocket carrying step 4's invariants. Warning: the boundary is enforced by capability (which tools each phase exposes), or it isn't enforced.

6. **Prototype the risky cell before committing.** Whichever structure you chose, run the 3–5 WORST-fitting sample tasks through a cheap prototype (even a prompt-only mock of the states). Structure failures show fast: the state machine hits a task needing a transition you didn't draw; the freeform agent skips the gate you assumed it would respect. Ten prototype runs cost less than one week of production surprises — and this is where the step-1 classification gets falsified or confirmed.

7. **Plan the migration path, because the distribution will drift.** Task mix changes; the structure that fit launch-quarter tasks won't fit next year's. Instrument which states/phases tasks actually traverse (`feedback-loop-instrumentation`): states never entered are removable; freeform pockets where 90% of runs follow one path are crystallization candidates (promote the path to a state); rising escape-hatch/escalation rates mean the structure is fighting the work. Re-run this skill's step 1 when the metrics say the shape moved.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "The model is smart enough — just let it figure out the steps" | Model capability doesn't enforce gates. The approval step the model "usually" respects is a control that fails open exactly under the adversarial or weird input where it mattered. |
| "A state machine is over-engineering for an AI agent" | For 80%-same-path tasks it's the opposite: less prompt, less cost, fewer failure modes than re-deriving the path every run. Over-engineering is building agency the task doesn't need. |
| "We'll start freeform and add structure when problems appear" | Reasonable IF the invariants (budgets, ledger, verification) ship on day one. What actually happens without this rule: the "temporary" freeform loop ships bare and the structure arrives via postmortem. |
| "Structure will limit what the agent can do" | That's its function — and the ceiling only binds if tasks genuinely exceed the stages. Check the sample (step 1) instead of asserting the exception. |
| "Our tasks are all unique, no skeleton exists" | Sketch ten actual paths first (step 1). "All unique" usually means "I haven't looked" — intake, verification, and delivery are near-universal skeleton bones. |
| "The hybrid is complex — pick one pure approach" | The hybrid IS the simple answer: each region gets the cheapest structure that works. Purity in either direction imports the other's failure mode at full price. |

## Red Flags

- A "state machine" whose states have no enforced tool restrictions — transitions by vibes, structure as documentation fiction.
- A freeform agent with safety-relevant steps described in the prompt but no structural gate — controls that fail open.
- No failure transitions drawn: every state's unhappy path is "hope."
- The task-sample step skipped; architecture chosen by team aesthetics ("we like agents" / "we like pipelines").
- 15+ states, several never entered in production metrics.
- Escalation/escape-hatch rate climbing quarter over quarter with nobody re-examining the structure.

## Verification

- [ ] Task sample (10+) with sketched paths exists and the step-1 classification is recorded — link the doc.
- [ ] The chosen structure is written down with states/phases, transitions (including failure transitions), and per-state tool scopes — reviewable artifact, not tribal knowledge.
- [ ] Gates and restrictions are enforced by capability (tool availability, permissions) — demonstrated by a test run where the agent attempts a forbidden action and structurally cannot.
- [ ] The worst-fitting sample tasks ran through a prototype; results and structure adjustments recorded.
- [ ] Freeform regions carry budgets, ledger, and verification — config linked.
- [ ] Traversal instrumentation live: which states/phases real runs visit is queryable.

## Example

Team building a contract-review agent (extract terms → flag risks → draft summary) argued agents-vs-pipeline for two weeks. Step 1 settled it in an afternoon: 12 sample contracts sketched — extraction and delivery paths identical across all 12 (state machine material), but risk-flagging paths varied wildly by contract type (freeform material). Hybrid shipped: `intake(validate format) → extract(structured, cheap model) → analyze(freeform loop, strong model, 8-iteration budget) → legal-gate(human checkpoint for any flag above severity 3 — enforced by the analyze phase having no send tools) → summarize → verify(all flagged clauses cited back to source text, checkable)`. The prototype step falsified one assumption: amendment documents broke the extract state (no base contract present) — a failure transition to "request-parent-document" was added before launch instead of after. Six months of traversal metrics then drove one migration: 92% of analyze-phase runs converged on the same 4-step pattern for NDAs specifically — that path crystallized into a fast NDA state (cost −60% on the highest-volume type), while the freeform pocket remained for everything else. The structure now earns re-review annually, triggered by the escalation-rate dashboard, not by architecture nostalgia.

## Related skills

- `agent-loop-design` — the internals of whichever structure wins.
- `human-in-the-loop-checkpoints` — gates as transitions, not suggestions.
- `runaway-cost-guardrails` — the freeform pockets' non-negotiable rails.
- `feedback-loop-instrumentation` — the traversal metrics driving migration.
- `architecture-decision-record` — where this decision and its drivers get recorded.
