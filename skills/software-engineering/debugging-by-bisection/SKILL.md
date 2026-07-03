---
name: debugging-by-bisection
description: >
  Use when a bug's cause is unknown and the search space is large — which
  commit broke it, which input triggers it, which config line matters.
  Halve the space repeatedly instead of reading everything. Triggers:
  "it worked last week", "which commit broke this", "git bisect",
  "can't find what's causing this", "worked before the upgrade".
---

# Debugging by Bisection

## When to use this skill
- A behavior regressed and some past version worked ("worked last release").
- A bug triggers on a big input/config and you don't know which part matters.
- NOT when you already have a stack trace pointing at a line — just go read the line.

## Prerequisites
- A **reliable, cheap check** that says pass/fail. Bisection on a flaky signal converges on garbage — stabilize the repro first (see `flaky-test-diagnosis`).
- For git bisection: one known-good and one known-bad ref.

## Workflow

1. **Pick the axis to bisect.** Time (commits), space (input data), or configuration (flags/env/deps). Choose the one where you have a confirmed good state and bad state.

2. **Script the check.** Even a 3-line script beats manual verification — you'll run it O(log n) times and manual checking is where mistakes creep in.
   ```bash
   # check.sh — exit 0 = good, 1 = bad, 125 = can't test (skip)
   ./build.sh || exit 125
   ./run_repro.sh
   ```

3. **Commits: use git bisect run.**
   ```bash
   git bisect start
   git bisect bad HEAD
   git bisect good v2.3.0
   git bisect run ./check.sh
   # ...git prints the first bad commit
   git bisect reset
   ```
   Exit code 125 skips unbuildable commits instead of poisoning the result.

4. **Inputs: delta-debug.** Cut the failing input in half; test each half. Failing half becomes the new input; recurse. When neither half fails alone, the bug needs both parts — minimize each half against the other (this is what `creduce`/`delta` automate for source files, and what shrinking does in property-based testing).

5. **Config: toggle halves.** Diff the working and broken config/env/dependency set, apply half the diff, test, recurse. `diff <(sort good.env) <(sort bad.env)` first — often the culprit is visible immediately.

6. **Confirm the culprit both directions.** Revert just the found commit/line on the bad state → passes. Apply just it to the good state → fails. If either check fails, your signal was unreliable — restart with a better check.

7. **Then diagnose.** Bisection gives *where*, not *why*. Read the culprit change and connect the mechanism to the symptom before writing the fix, or you'll fix the coincidence instead of the cause.

## Common pitfalls
- Bisecting with a flaky test. One wrong answer sends `git bisect` down the wrong half permanently. Run the check 3–5× per step if there's any doubt.
- Forgetting `git bisect reset` — you're left detached on an ancient commit and the next hour is confusing.
- Bisecting when the good state is misremembered ("pretty sure it worked in March"). Verify the good endpoint actually passes before starting, or you'll bisect the wrong range.
- Stopping at the commit. A 900-line commit still needs input/section bisection *within* it — check out the culprit and bisect its hunks with `git stash`/partial reverts.

## Example
API latency regressed sometime in ~600 commits. `check.sh` runs a 30-request benchmark, exits 1 if p95 > 200ms, 125 if build fails. `git bisect run ./check.sh` finishes in 10 automated steps (~9 min), landing on a commit that swapped a connection pool default from 20 to 5. Confirmed: reverting only that line on HEAD restores p95. Fix targets the pool size, not the symptom.

## Related skills
- `flaky-test-diagnosis` — stabilizing the pass/fail signal bisection depends on.
- `production-debugging` — when there's no repro to bisect, only live symptoms.
- `refactor-safely` — small commits are what make step 3 land on a readable culprit.
