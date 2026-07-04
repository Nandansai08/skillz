---
name: context-window-loop-management
description: >
  Use when a long-running agent loop must decide what to keep, drop, or
  summarize each iteration so context stays within budget without losing
  the state that matters. Triggers: "context window filling up", "agent
  forgets earlier steps", "summarize agent history", "context compaction",
  "long-running agent memory", "token budget per iteration". NOT for
  cross-session persistent memory stores (different problem: retrieval),
  and NOT for single-call prompt-size optimization.
---

# Context Window Loop Management

## Overview

A loop that naively appends every observation dies one of two deaths: context overflow, or quality decay as the signal drowns in its own history. This skill designs the per-iteration context diet — what survives verbatim, what compresses, what drops — so iteration 40 reasons as well as iteration 4.

## When to Use

- Designing any agent loop expected to exceed a handful of iterations.
- An agent's quality degrades over long runs (forgets the goal, re-asks answered questions, contradicts earlier decisions).
- Compaction already exists but causes the failures it was meant to prevent (`tool-use-loop-debugging` step 3's dissolved-ledger case).

**When NOT to use:**
- Cross-session memory ("remember the user's preferences next week") — that's a retrieval/persistence design, not per-iteration diet.
- One-shot prompts that are merely large — that's prompt engineering and RAG, no loop dimension.

## Prerequisites

- The loop's state schema (`agent-loop-design` step 2) — you can't design retention tiers for state that was never named.
- Token/cost telemetry per iteration (`feedback-loop-instrumentation`) — diet design without measurements is guessing at the menu.

## The Workflow

1. **Tier the loop's information by retention class — the core design move:**
   - **Pinned (never compressed, never dropped):** the goal, hard constraints, the attempt ledger, key decisions with their one-line whys, budget status. Small by construction — measured in hundreds of tokens, defended absolutely.
   - **Working set (verbatim, sliding):** the last 2–4 iterations' full detail — recent context is what the next action actually conditions on.
   - **Compressible history:** older iterations' plans/observations — summarized on a schedule (step 3).
   - **Droppable exhaust:** raw tool output already distilled into observations (the 30k-token log whose lesson is one line), superseded file reads, dead-end details beyond their ledger entry.
   The design failure behind most long-run decay is tier confusion: ledger treated as compressible, raw logs treated as working set.

2. **Distill at the moment of observation, not at compaction time.** When a tool returns 20k tokens, extract the decision-relevant summary IMMEDIATELY (while the loop knows what question the output was answering) and keep a pointer to the raw artifact (file path, trace ID) for re-fetch if needed. Deferred distillation compresses blind — the compactor at iteration 30 no longer knows which line of the iteration-6 log mattered. Warning: keep error outputs closer to verbatim than success outputs — errors front-load diagnostic detail that summaries routinely amputate (`tool-use-loop-debugging` step 2's truncated-stderr classic).

3. **Choose the compaction trigger and shape:**

   ```
   Context usage crosses threshold (e.g. 70%)?
        |                          |
        no — continue              yes
                                   |
                                   v
              Natural phase boundary nearby (plan done,
              subtask complete)?
                    |                    |
                   yes                   no
                    |                    |
                    v                    v
           Compact at the boundary   Compact now, but
           (phase summaries are      re-derive from tiers:
           the cleanest compression  pinned survives verbatim,
           points — agent-loop-      working set survives,
           design step 5's seams)    history → structured
                                     summary
   ```

   Compress into STRUCTURE, not prose: "Phase: diagnosis. Findings: [3 bullets]. Ruled out: [2 items]. Open: [1 question]" — structured summaries preserve retrievability; narrative summaries preserve mood.

4. **Make the summary auditable against loss.** After each compaction, the loop should still be able to answer: what is the goal? what has been tried? what was decided and why? If any answer degraded, the tiering is wrong — promote the lost class to pinned. Cheap mechanical check: the attempt-ledger's entry count must never decrease across a compaction (the exact failure that turns compaction into perseveration fuel — `tool-use-loop-debugging` step 3).

5. **Prefer subtask isolation over heroic compression for big work units.** A subtask needing 80k tokens of exploration (read a whole module, scan a dataset) runs in its own scoped context — sub-agent or fresh call — returning only its distilled result to the parent loop. The parent's context carries the CONCLUSION, not the expedition. This is the highest-leverage pattern in the skill: it converts context management from compression engineering into interface design (what does the parent actually need back?).

6. **Budget per-iteration and watch the shape.** Set an expected tokens-per-iteration envelope; instrument actual usage by tier (`feedback-loop-instrumentation` step 3's state-size snapshot). Two trend alarms: monotonic growth per iteration (something isn't being dropped — the leak pattern), and pinned-tier creep (everything "important" migrating to pinned until it's the whole window — pinned above ~15% of budget means the tiering has stopped tiering).

7. **Test degradation explicitly with a long-horizon eval.** Add to the agent's eval suite (`eval-loop-for-agents`) tasks requiring 20+ iterations, scored on late-run behaviors specifically: goal recall at iteration N, no re-attempting of ledgered failures, decisions consistent with early constraints. Compaction bugs are invisible in short evals by construction — the suite must contain runs long enough to compact several times.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "The model has a 200k window — we don't need management" | Long windows delay overflow, not decay: retrieval quality over huge undifferentiated histories drops (needle-in-haystack costs), and you pay the full window's tokens every iteration. The diet is a cost AND quality tool at any window size. |
| "Just summarize everything older than N turns" | Uniform summarization is tier-blind — it compresses the attempt ledger and the goal with the same enthusiasm as dead logs, and the loop starts retrying its own failures. Tiers first, then schedules. |
| "The agent will keep what's important" | Untiered, the agent keeps what's recent and vivid. Importance is a design input (pinned tier), not an emergent property of appending. |
| "We'll add compaction when we hit the limit" | The first overflow event is a production incident with a truncated-mid-task agent. The tier design costs an hour at design time; retrofitted under incident pressure it costs the ledger (see the classic). |
| "Summarization loses information, so we avoid it entirely" | Keeping everything IS a lossy strategy — it loses attention. The choice isn't lossless vs lossy; it's designed loss vs accidental loss. |
| "Sub-agents are complexity we don't need" | For big explorations, the sub-task pattern is the SIMPLER system: parent context stays clean, and the alternative is compression heroics inside one window. |

## Red Flags

- Context usage growing monotonically per iteration with no compaction events in the trace.
- The agent re-asking answered questions or re-attempting ledgered failures after a compaction event (loss symptom, tier bug).
- Attempt-ledger entry count decreasing across compaction — the mechanical tripwire firing.
- Raw multi-thousand-token tool outputs sitting verbatim in iteration 25's context.
- Compaction summaries that are narrative paragraphs instead of structured fields.
- Pinned tier above ~15% of the window and climbing.
- Eval suite contains no task longer than 10 iterations.

## Verification

- [ ] Retention tiers documented in the loop's design (what's pinned/working/compressible/droppable) — link.
- [ ] Distill-at-observation implemented: a trace shows a large tool output reduced to a summary + raw pointer in the same iteration.
- [ ] Compaction preserves the ledger: a trace across a compaction event shows attempt-ledger count non-decreasing.
- [ ] Post-compaction audit passes: goal, attempts, and decisions answerable from the compacted context — sample trace attached.
- [ ] Per-tier token telemetry visible on the dashboard — link.
- [ ] Long-horizon eval task (20+ iterations, ≥2 compactions) exists in the suite and passes — run linked.

## Example

Research agent (multi-document analysis, 30–60 iterations typical) degrading badly past iteration ~25: re-reading documents it had already processed, contradicting its own earlier extraction decisions. Trace review: naive append-everything until 85% capacity, then a single prose mega-summary — which had been dissolving both the ledger and the per-document decisions (the audit question "what was decided about doc 7?" was unanswerable post-compaction). Redesign: tiers — pinned = research question + extraction schema + per-document verdict table (one line each) + attempt ledger; working set = last 3 iterations; documents processed via subtask isolation (each doc analyzed in a scoped call returning a structured verdict — the parent never saw raw document text at all, the step-5 pattern retrofitted). Compaction moved to phase boundaries (per-document completion) with structured summaries. Results: context per iteration flattened at ~30% of window (from monotonic growth), re-read rate zero across a 55-iteration benchmark run, and cost per task −44% (the parent no longer re-paying for document text every iteration). The long-horizon eval added afterward caught one regression two months later: a prompt refactor accidentally moved the verdict table out of the pinned block — flagged by the goal-recall check at iteration 30, fixed before production noticed.

## Related skills

- `agent-loop-design` — the state schema these tiers organize.
- `tool-use-loop-debugging` — the memory-loss pathologies this design prevents.
- `feedback-loop-instrumentation` — the telemetry the diet is tuned against.
- `eval-loop-for-agents` — the long-horizon tasks that test it.
- `note-synthesis` — the same distill-into-claims move, human edition.
