---
name: regression-test-from-bug
description: >
  Use when fixing a bug — turn the bug report into a failing test before
  writing the fix, so the bug can never silently return. Triggers: "fix this
  bug", "write a regression test", "reproduce this issue in a test",
  "make sure this doesn't happen again", closing any bug ticket.
---

# Regression Test From Bug

## When to use this skill
- Any bug fix, before the fix is written.
- Post-incident: converting an outage's trigger into a permanent test.
- NOT for hypothetical bugs — that's `edge-case-enumeration` during normal test writing.

## Prerequisites
- A bug report with enough to reproduce: input, expected, actual. If reproduction is unknown, that's the first job — no repro, no fix, only a guess.

## Workflow

1. **Reduce the report to a minimal trigger.** Strip the reported scenario to the smallest input that still misbehaves (delta-debugging by hand: halve the input, keep the failing half). The minimal trigger names the real bug — "crashes on orders with a deleted product" beats "crashes on Karen's order from Tuesday."

2. **Write the test at the lowest layer that exhibits the bug.** If the broken logic is a pure function, unit test it there — don't write a browser test because the *report* came through the UI. Drop a layer until the bug disappears; test one layer above that point.

3. **Name it after the bug, link the ticket.**
   ```python
   def test_discount_not_applied_twice_on_retry():  # regression: #4412
   ```
   The link matters at 3am two years later when someone wants to delete the "weird" test.

4. **Run it and watch it fail — for the right reason.** The assert must fail with the *reported* wrong behavior, not an unrelated error in setup. A regression test that fails for the wrong reason will pass for the wrong reason after any change.

5. **Now write the fix.** Test goes green. Nothing else in the suite goes red (if it does, either the fix is wrong or a test was frozen around the bug — decide which, explicitly).

6. **Sweep for siblings.** The bug had a cause (unchecked null, race, off-by-one). Grep for the same pattern elsewhere in the codebase; each hit gets the same treatment or a ticket. Bugs cluster — the reporter found one instance of a class.

7. **Ask what let it escape.** One sentence in the PR: which missing test category (edge case? integration seam? concurrency?) would have caught this class, and whether it's worth adding systematically. This is the step that improves the suite instead of just patching it.

## Common pitfalls
- Fix first, test after. The test then gets written to pass against the fixed code — you never verified it detects the bug. Order is the whole point.
- Testing the fix's implementation ("retry counter equals 1") instead of the reported behavior ("charge happens once"). The former breaks on refactor; the latter guards the contract.
- A regression test reproducing the whole user scenario through five services. Slow, flaky, and it'll be quarantined within a year — which un-guards the bug. Minimize per step 2.
- Skipping step 4 under time pressure — the most common way regression suites fill with tests that assert nothing.
- Freezing the *symptom* when the mechanism is elsewhere: asserting the UI shows no error while the double-charge still happens in the ledger.

## Example
Report: "customer charged twice." Step 1 minimized to: retry after timeout re-runs charge because the idempotency key was regenerated per attempt. Step 2: bug exhibits at the `PaymentClient` layer — unit test with a fake gateway asserting one charge across a timeout-retry. Step 4: fails showing 2 charges. Fix: key derived from order ID, not per-call UUID. Step 6 grep for `uuid4()` near retry decorators found the same pattern in refunds — second ticket filed. Step 7: added a "retry-safety" section to the review checklist.

## Related skills
- `debugging-by-bisection` — when reproduction is the hard part.
- `unit-test-design` — the structure/naming rules the regression test follows.
- `blameless-postmortem` — step 7 at incident scale.
