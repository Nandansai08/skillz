---
name: tool-use-loop-debugging
description: >
  Use when an agent loop misbehaves — infinite loops, the same failing tool
  call repeated, thrashing between two states, silent stalls — and you need
  the diagnostic method. Triggers: "agent is stuck in a loop", "keeps
  calling the same tool", "agent thrashing", "burned tokens doing nothing",
  "agent repeats the same fix", "loop never terminates".
---

# Tool-Use Loop Debugging

## When to use this skill
- An agent is observably looping: repeated calls, oscillation, no progress against cost.
- Post-hoc analysis of a runaway session's transcript.
- NOT for designing the loop (`agent-loop-design`) or setting the caps that should have fired (`runaway-cost-guardrails`) — though this skill's findings routinely feed both.

## Prerequisites
- The transcript/trace of the misbehaving run (full tool calls + results + the agent's intermediate reasoning where available) — loop debugging without the trace is speculation; `feedback-loop-instrumentation` is the prerequisite skill if no trace exists.

## Workflow

1. **Classify the pathology from the trace — the four species:**
   - **Perseveration:** the same call (or semantic equivalent) repeated despite failing — same tool, same-ish args, same error, again.
   - **Two-state thrash:** A undoes B undoes A (the edit that breaks lint, the lint-fix that breaks the test; the two config values toggled forever).
   - **Drift loop:** progress-shaped motion with no convergence — new actions each time, none advancing the actual goal (subtask spawning subtasks, exploration that never returns).
   - **Stall:** the loop "runs" but acts on nothing (empty tool results accepted silently, the agent narrating plans without executing them).
   Diff consecutive iterations to classify: identical = perseveration; alternating = thrash; novel-but-goalless = drift; act-free = stall. The species determines the fix family.

2. **Find what the agent couldn't SEE — the majority root cause:** most repetition is an observability gap, not a reasoning gap. Check the trace for: error messages truncated before the informative part (the agent retries because the failure looked transient — it couldn't read why it failed); tool results summarized into uselessness; success/failure ambiguity (exit code lost, the "did it apply?" question unanswerable); and state the agent assumed but never verified (`agent-loop-design` step 4's assumed-observation sin, now presenting as its symptom). Fix the information diet first — a perseverating agent shown its actual error usually stops perseverating.

3. **Check what the agent couldn't REMEMBER:** does the failed attempt survive into the next iteration's context, recognizably? The classic mechanics: context compaction/summarization dropped the attempt history (the agent literally doesn't know it tried this — `context-window-loop-management`'s trade-off biting); the attempt recorded but semantically invisible (buried in 40k tokens of logs); or no attempt ledger existed at all. The fix is structural: an explicit, compact, always-visible attempt ledger ("tried: pin to 2.1 — failed same error; tried: clear cache — no effect") — the single highest-value patch in this genre.

4. **For thrash specifically, surface the hidden constraint conflict:** two-state oscillation almost always means two objectives that genuinely conflict (the lint rule vs the test's expectation; requirement A vs requirement B) and the agent is satisfying whichever it saw last. The trace shows it optimizing each side alternately. Fixes in order: make the conflict visible to the agent in one place (both constraints stated together — agents resolve visible conflicts and oscillate on invisible ones); if genuinely irreconcilable, that's an escalation condition, not a solvable loop (`human-in-the-loop-checkpoints` — the agent's job becomes REPORTING the conflict).

5. **Check the loop's own plumbing before blaming the model:** tool bugs that return success on failure (the agent "verified" against a lying tool); nondeterministic tools making identical calls reasonable (the flaky test the agent legitimately re-runs — `flaky-test-diagnosis` for the tool itself); prompts that accidentally instruct repetition ("always retry on failure" without a cap); and temperature/sampling turning a decisive prompt into a dithering one. A model-blame diagnosis is only earned after the environment is acquitted.

6. **Install the structural anti-loop machinery the failure revealed:**
   - **Repetition detection:** hash/fingerprint each action (tool + normalized args); N repeats triggers forced strategy change or exit — cheap, mechanical, catches perseveration AND semantically-identical "different" attempts (normalize enough that reworded-identical matches).
   - **Progress metrics:** a per-iteration "distance to goal" signal where definable (tests passing count, TODO items closed) — flat over K iterations = drift alarm (`convergence-criteria-design` supplies the metric).
   - **The reflect-gate:** before each action, the forced question "how does this differ from previous attempts, and why will it work when they didn't?" — prompted-for explicitly, it converts the attempt ledger into behavior change.
   - Budget backstops verified live (`runaway-cost-guardrails` — this incident is the test they either passed or failed).

7. **Re-run the failing scenario and verify the fix mechanically:** the original stuck case now exits (fixed, failed-with-report, or escalated — any is acceptable; spinning is not) within budget; then regression-pin it into the agent's eval set (`eval-loop-for-agents` — every debugged loop failure is a permanent eval case, exactly `regression-test-from-bug`'s move at agent scale).

## Common pitfalls
- Prompt-tweaking first: "I told it not to repeat itself" — treating an information/memory gap as a persuasion problem. Steps 2–3 (what could it see, what could it remember) come before any prompt surgery, and usually end the investigation.
- Trace-free debugging: reasoning about what the agent "must be doing" from its final output — the transcript exists (or `feedback-loop-instrumentation` is your actual next step); read it.
- Blaming the model for a lying tool: the environment's bugs wear the agent's face (step 5's acquittal ordering).
- Repetition detection that's too literal: exact-match hashing while the agent rephrases the same failing attempt — normalize (strip whitespace, sort args, hash the semantic core) or the detector watches the thrash politely.
- Fixing the instance, not the class: this loop unstuck, the machinery (ledger, detector, reflect-gate) uninstalled — the next variant re-runs the whole investigation (step 6 is the deliverable; step 7's eval pin is the receipt).
- Removing the budget cap "now that it's fixed": the cap is the backstop for the failure mode you haven't met yet.

## Example
Incident: coding agent burned 40k tokens re-attempting the same dependency fix — 11 iterations, same `pip install` variant, same resolution error. Trace classification: perseveration (consecutive-iteration diff near-identical). Step 2's finding, the actual root cause: the tool wrapper truncated stderr at 500 chars — the resolver's "package X requires Y<2.0" explanation sat at char 800; the agent saw only "ERROR: ResolutionImpossible" and reasonably kept trying variants of the same install. Step 3's finding stacked on top: context summarization at iteration 8 had compressed attempts 1–7 into "attempted several installation approaches" — the ledger dissolved exactly when it mattered. Fixes: stderr returned in full (tail-truncated, head preserved — errors front-load); compact attempt ledger (action-hash + one-line outcome) pinned outside summarization reach; repetition detector at 2 same-hash attempts → forced strategy change, 3 → exit-with-report. Re-run: iteration 2 read the constraint, pinned Y correctly, done in 3 iterations. The case entered the eval set; the detector's first quarter caught two novel perseverations early — both, on inspection, ALSO observability gaps (a lying exit code, a swallowed timeout), confirming the genre's rule: the agent that repeats itself usually couldn't see or couldn't remember.

## Related skills
- `feedback-loop-instrumentation` — the traces this debugging requires.
- `agent-loop-design` — the design layer these findings patch.
- `runaway-cost-guardrails` — the backstop this incident stress-tested.
- `eval-loop-for-agents` — where the fixed case gets pinned.
- `retry-and-backoff-strategy` — legitimate retries vs perseveration: the boundary.
