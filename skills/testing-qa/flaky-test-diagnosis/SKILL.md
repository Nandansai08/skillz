---
name: flaky-test-diagnosis
description: >
  Use when a test passes sometimes and fails sometimes — locally vs CI, in
  isolation vs in suite — and you need the root cause, not a retry
  band-aid. Triggers: "flaky test", "passes locally fails in CI", "fails
  randomly", "only fails when run with other tests", "intermittent test
  failure". NOT for consistently-failing tests (that's a plain bug — debug
  it directly) and NOT for suite-level e2e flakiness policy (see
  e2e-test-triage).
---

# Flaky Test Diagnosis

## Overview

A flake has exactly one of a short list of causes — interference, time, ordering, randomness, environment — and a reproduction loop finds which. Retry-until-green hides the cause, triples CI cost, and occasionally hides a real race in the code under test.

## When to Use

- A test's outcome varies with no code change.
- CI has retries enabled and you want to actually fix the tests hiding behind them.

**When NOT to use:**
- Consistent failures — ordinary debugging.
- Deciding suite-wide e2e policy (quarantine rules, retry budgets) — `e2e-test-triage`.

## Prerequisites

- Ability to run the test repeatedly and control test order locally.
- The failure output from at least one real failure (assert message, not just "failed").

## The Workflow

1. **Reproduce the flake rate first.** Run the test alone in a loop:
   ```bash
   pytest path/to/test.py::test_name --count=200 -x      # pytest-repeat
   go test -run TestName -count=200 -failfast
   ```

   ```
   Fails alone?  ──yes──>  nondeterminism inside the test (step 3)
        │no
        v
   Fails only in suite?  ──yes──>  inter-test interference (step 2)
        │no
        v
   Fails only in CI  ──────────>  environment delta (step 4)
   ```

2. **Interference: find the poisoning test.** Run in the recorded failing order (`pytest -p no:randomly` with the CI seed, or the CI log's order). Then bisect the preceding tests: run half of them before the victim, recurse. Usual culprits: shared DB rows without cleanup, module-level caches, mutated globals/env vars, leaked mock patches, singleton state. Fix the *polluter*, not the victim — the victim is just the first test honest enough to notice.

3. **Nondeterminism inside the test — check the usual five in order:**
   - **Time:** real clocks, `sleep(0.1)`-and-hope, timeouts near actual duration. Fix: inject/freeze the clock; replace sleeps with condition-waits.
   - **Async/ordering:** awaiting one of several tasks, assuming callback order. Fix: await completion signals explicitly.
   - **Unordered collections:** asserting on dict/set/query order that isn't guaranteed. Fix: sort before assert. (Watch for hash randomization — `PYTHONHASHSEED`.)
   - **Randomness:** unseeded RNG in code or fixtures/faker. Fix: seed it in the test.
   - **External dependencies:** real network, filesystem races, shared test DB with parallel workers. Fix: isolate per-worker or double it.

4. **CI-only: diff the environments.** Suspects ranked by frequency: fewer CPUs (slower → timeouts fire, races reorder), different timezone/locale, parallelism enabled in CI only, missing service warm-up, container memory limits, different DB version. Reproduce locally by imitating: `docker run --cpus=1`, `TZ=UTC`, run with the CI's parallelism flags.

5. **Prove the fix.** Rerun the loop from step 1 — 200+ iterations green, and in suite order. The flake was 1-in-50; ten green runs prove nothing.

6. **Institutionalize.** Quarantine markers for known flakes with owner + deadline (`@pytest.mark.flaky_quarantine(issue=4412)`), a CI job that runs new tests 20× before merge, and delete retries-as-policy: retries convert flakes from visible failures into invisible 3× CI cost and masked real races.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "Just add a retry — it passes the second time" | Retries hide the cause, and sometimes the cause is a real race in production code. The example below found a double-issuing bug behind exactly this reasoning. |
| "A sleep(5) fixed it" | It moved the race threshold; loaded CI will find it again at sleep(5)+ε. Condition-waits fix the race; sleeps schedule its return. |
| "It's the test's fault — patch cleanup into the failing test" | In interference cases the failing test is the victim. Patching it leaves the polluter poisoning the next test alphabetically. |
| "Ran it ten times after the fix, all green" | The flake rate was 1-in-50. Ten runs is a coin flip's worth of evidence; the 200-loop is the proof standard. |
| "Quarantine it for now, we'll fix it later" | Quarantine without owner+deadline is deletion with extra steps — and the coverage it provided is silently gone the whole time. |
| "Flaky tests are just a fact of life at scale" | Flake RATE is a choice. Teams holding <0.5% do it with exactly this diagnosis loop plus the new-test 20× gate; acceptance is how 4% happens. |

## Red Flags

- CI config contains blanket `retries: N` with no flake-rate tracking.
- Sleeps added in commits titled "fix flaky test."
- The same test quarantined for months, no owner, no deadline.
- Flakes "fixed" with no reproduction loop before or after.
- A test that fails only in suite, "fixed" by editing the test itself.
- Rising CI duration from silent retry inflation.

## Verification

- [ ] Flake reproduced and rate measured (loop output: N failures / M runs) — before the fix.
- [ ] Root cause named from the taxonomy (interference/time/ordering/randomness/environment) — in the fix's PR.
- [ ] For interference: the polluter identified and fixed — not the victim patched.
- [ ] Fix proven: 200+ green iterations alone AND in suite order — output linked.
- [ ] If the cause implicated production code (real race), a production fix + regression test shipped (see regression-test-from-bug) — linked.

## Example

`test_order_expiry` failed ~1/30 CI runs, never locally. Step 1: green ×500 alone. Step 4: CI runs 4-way parallel against one Postgres; with local parallelism enabled, repro at 1/20. Root cause: two workers sharing a sequence-generated "unique" order code with a check-then-insert race — which the *application* also had. Fix: unique constraint + upsert in app code, per-worker schemas in tests. The flaky test had found a production bug that had been intermittently double-issuing codes.

## Related skills

- `debugging-by-bisection` — the polluter search in step 2 is bisection.
- `e2e-test-triage` — flakiness policy at the end-to-end layer.
- `test-doubles-choice` — isolating external dependencies found in step 3.
- `regression-test-from-bug` — when the flake exposes a production race.
