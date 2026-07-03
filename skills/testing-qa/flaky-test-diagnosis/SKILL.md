---
name: flaky-test-diagnosis
description: >
  Use when a test passes sometimes and fails sometimes — locally vs CI, in
  isolation vs in suite — and you need the root cause, not a retry band-aid.
  Triggers: "flaky test", "passes locally fails in CI", "fails randomly",
  "only fails when run with other tests", "intermittent test failure".
---

# Flaky Test Diagnosis

## When to use this skill
- A test's outcome varies with no code change.
- CI has retries enabled and you want to actually fix the tests hiding behind them.
- NOT for a test that fails consistently — that's a plain bug; debug it directly.

## Prerequisites
- Ability to run the test repeatedly and control test order locally.
- The failure output from at least one real failure (assert message, not just "failed").

## Workflow

1. **Reproduce the flake rate first.** Run the test alone in a loop:
   ```bash
   pytest path/to/test.py::test_name --count=200 -x      # pytest-repeat
   go test -run TestName -count=200 -failfast
   ```
   - Fails alone → nondeterminism inside the test (go to step 3).
   - Never fails alone → inter-test interference (go to step 2).
   - Only fails in CI → environment delta (go to step 4).

2. **Interference: find the poisoning test.** Run in the recorded failing order (`pytest -p no:randomly` with the CI seed, or the CI log's order). Then bisect the preceding tests: run half of them before the victim, recurse. Usual culprits: shared DB rows without cleanup, module-level caches, mutated globals/env vars, leaked mock patches (`patch` without context manager), singleton state. Fix the *polluter*, not the victim — the victim is just the first test honest enough to notice.

3. **Nondeterminism inside the test — check the usual five in order:**
   - **Time:** real clocks, `sleep(0.1)`-and-hope, timeouts near actual duration, tests near midnight/month-end. Fix: inject/freeze the clock; replace sleeps with condition-waits.
   - **Async/ordering:** awaiting one of several tasks, assuming callback order, races between setup thread and act. Fix: await completion signals explicitly.
   - **Unordered collections:** asserting on dict/set/query order that isn't guaranteed. Fix: sort before assert or assert on sets. (Watch for hash randomization — Python `PYTHONHASHSEED`.)
   - **Randomness:** unseeded RNG in code or fixtures/faker. Fix: seed it in the test.
   - **External dependencies:** real network, real filesystem races, shared test DB with parallel workers. Fix: isolate per-worker (unique schema/tmpdir) or double it.

4. **CI-only: diff the environments.** Suspects ranked by frequency: fewer CPUs (slower → timeouts fire, races reorder), different timezone/locale, parallelism enabled in CI only, missing service warm-up, container memory limits, different DB version. Reproduce locally by imitating: `docker run --cpus=1`, `TZ=UTC`, run with the CI's parallelism flags.

5. **Prove the fix.** Rerun the loop from step 1 — 200+ iterations green, and in suite order. A flake "fixed" without a repro loop is a flake postponed.

6. **Institutionalize.** Quarantine markers for known flakes with owner + deadline (`@pytest.mark.flaky_quarantine(issue=4412)`), a CI job that runs new tests 20× before merge, and delete retries-as-policy: retries convert flakes from visible failures into invisible 3× CI cost and masked real races.

## Common pitfalls
- Adding `sleep(5)` as the fix. It moves the race threshold; loaded CI will find it again. Wait on conditions, not clocks.
- Blaming the victim test in interference cases — patching cleanup into the failing test leaves the polluter poisoning the next victim.
- Marking flaky-and-retry forever. A quarantine without a deadline is a deletion with extra steps.
- Fixing without reproducing: the flake was 1-in-50; your "fix" plus 10 green runs proves nothing. Loop it.
- Ignoring flaky tests as "just test problems." A race in a test is frequently a real race in the code under test — check before suppressing.

## Example
`test_order_expiry` failed ~1/30 CI runs, never locally. Step 1: green ×500 alone. Step 4: CI runs 4-way parallel against one Postgres; with local parallelism enabled, repro at 1/20. Root cause: two workers sharing a sequence-generated "unique" order code with a check-then-insert race — which the *application* also had. Fix: unique constraint + upsert in app code, per-worker schemas in tests. The flaky test had found a production bug that had been intermittently double-issuing codes.

## Related skills
- `debugging-by-bisection` — the polluter search in step 2 is bisection.
- `e2e-test-triage` — flakiness policy at the end-to-end layer.
- `test-doubles-choice` — isolating external dependencies found in step 3.
