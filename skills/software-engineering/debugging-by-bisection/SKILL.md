---
name: debugging-by-bisection
description: >
  Use when a bug's cause is unknown and the search space is large — which
  commit broke it, which input triggers it, which config line matters —
  and the space can be halved repeatedly. Triggers: "it worked last week",
  "which commit broke this", "git bisect", "can't find what's causing
  this", "worked before the upgrade". NOT when a stack trace already
  points at a line (just read the line), and NOT for live-production
  diagnosis without a repro (see production-debugging).
---

# Debugging by Bisection

## Overview

Halving a search space beats reading it: log₂(600 commits) is 10 checks, not 600. Bisection converts "somewhere in all this" into a mechanical procedure — provided the pass/fail signal is reliable, which is where the method lives or dies.

## When to Use

- A behavior regressed and some past version worked ("worked last release").
- A bug triggers on a big input/config and you don't know which part matters.

**When NOT to use:**
- A stack trace or error already names the location — go read it.
- No repro exists, only live symptoms — `production-debugging` first; bisection needs a check you can run at will.

## Prerequisites

- A **reliable, cheap check** that says pass/fail. Bisection on a flaky signal converges on garbage — stabilize the repro first (see `flaky-test-diagnosis`).
- For git bisection: one known-good and one known-bad ref — and verify the "good" endpoint actually passes before starting; a misremembered good ref bisects the wrong range to a confident wrong answer.

## The Workflow

1. **Pick the axis to bisect.** Time (commits), space (input data), or configuration (flags/env/deps). Choose the one where you have a confirmed good state and bad state.

2. **Script the check.** Even a 3-line script beats manual verification — you'll run it O(log n) times and manual checking is where mistakes creep in.
   ```bash
   # check.sh — exit 0 = good, 1 = bad, 125 = can't test (skip)
   ./build.sh || exit 125
   ./run_repro.sh
   ```
   If the check has any flakiness, run it 3–5× per step inside the script — one wrong answer sends the whole search down the wrong half permanently.

3. **Commits: use git bisect run.**
   ```bash
   git bisect start
   git bisect bad HEAD
   git bisect good v2.3.0
   git bisect run ./check.sh
   # ...git prints the first bad commit
   git bisect reset    # ALWAYS — or you're left detached on an ancient commit
   ```
   Exit code 125 skips unbuildable commits instead of poisoning the result.

4. **Inputs: delta-debug.** Cut the failing input in half; test each half. Failing half becomes the new input; recurse. When neither half fails alone, the bug needs both parts — minimize each half against the other (this is what `creduce`/`delta` automate for source files, and what shrinking does in property-based testing).

5. **Config: toggle halves.** Diff the working and broken config/env/dependency set, apply half the diff, test, recurse. `diff <(sort good.env) <(sort bad.env)` first — often the culprit is visible immediately.

6. **Confirm the culprit both directions.** Revert just the found commit/line on the bad state → passes. Apply just it to the good state → fails. If either check fails, your signal was unreliable — restart with a better check.

7. **Then diagnose.** Bisection gives *where*, not *why*. Read the culprit change and connect the mechanism to the symptom before writing the fix, or you'll fix the coincidence instead of the cause. A 900-line culprit commit still needs bisection *within* it — check it out and bisect its hunks with partial reverts.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "Faster to just read the recent commits" | Ten commits, maybe. Six hundred, no — and "recent" assumes the regression is recent, which is exactly what you don't know. The scripted bisect runs unattended while you do other work. |
| "The check is a bit flaky but mostly right" | One wrong verdict poisons the entire remaining search — bisection amplifies signal unreliability into confident wrong answers. Stabilize first or loop the check per step. |
| "I'm pretty sure it worked in the March release" | "Pretty sure" bisects the wrong range. Verifying the good endpoint costs one check run; skipping it can cost the whole session. |
| "Found the commit, that's the bug — ship the revert" | The commit is where, not why. Reverts without mechanism understanding routinely revert the co-traveler and leave the cause — step 6's two-direction confirmation is ten minutes. |
| "Manual checking is fine, it's only ~10 steps" | Ten manual checks at 2am produce at least one misread verdict. The script exists because humans are the flaky component. |
| "Can't bisect — the old commits don't build" | Exit 125 exists precisely for this; git skips and narrows around unbuildable regions. |

## Red Flags

- Bisecting with a check that's failed to reproduce consistently even once.
- No `git bisect reset` — subsequent confusion about "why is the repo in a weird state."
- Culprit accepted without the revert-and-apply confirmation (step 6).
- Fix shipped that doesn't mention the mechanism connecting culprit to symptom.
- "Bisected" by reading commit messages and guessing.
- The good endpoint never verified before starting.

## Verification

- [ ] Check scripted, with exit codes 0/1/125 — script committed or attached.
- [ ] Good endpoint verified passing before the search started (run output).
- [ ] Culprit confirmed both directions: revert-on-bad passes AND apply-on-good fails — both outputs recorded.
- [ ] Mechanism stated: one sentence connecting the culprit change to the symptom, in the fix's PR.
- [ ] `git bisect reset` run (repo on the original branch).

## Example

API latency regressed sometime in ~600 commits. `check.sh` runs a 30-request benchmark, exits 1 if p95 > 200ms, 125 if build fails. `git bisect run ./check.sh` finishes in 10 automated steps (~9 min), landing on a commit that swapped a connection pool default from 20 to 5. Confirmed: reverting only that line on HEAD restores p95. Fix targets the pool size, not the symptom.

## Related skills

- `flaky-test-diagnosis` — stabilizing the pass/fail signal bisection depends on.
- `production-debugging` — when there's no repro to bisect, only live symptoms.
- `refactor-safely` — small commits are what make step 3 land on a readable culprit.
