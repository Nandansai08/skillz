---
name: regression-test-from-bug
description: >
  Use when fixing a reported bug — any bug, before the fix is written — so
  the defect can never silently return. Triggers: "fix this bug", "write a
  regression test", "reproduce this issue", closing any bug ticket,
  post-incident test hardening. NOT for enumerating hypothetical edge cases
  during normal test writing (see edge-case-enumeration), and NOT for
  building first-time coverage on untested legacy code (see
  legacy-code-first-contact).
---

# Regression Test From Bug

## Overview

Turn every bug report into a failing test before writing the fix. The test proves the bug is understood, proves the fix works, and guards the behavior forever — a fix without it is a guess with a commit message.

## When to Use

- Any bug fix, before the fix is written.
- Post-incident: converting an outage's trigger into a permanent test.
- A "fixed" bug just came back — the missing regression test IS the finding.

**When NOT to use:**
- Hypothetical bugs found by inspection during test writing — that's `edge-case-enumeration` feeding `unit-test-design`.
- Untested legacy code needing a general safety net before changes — that's `legacy-code-first-contact` (characterization tests freeze current behavior; this skill freezes *correct* behavior against a specific defect).

## Prerequisites

- A bug report with enough to reproduce: input, expected, actual. If reproduction is unknown, that's the first job — no repro, no fix, only a guess (`debugging-by-bisection` when the trigger is buried in a large input or commit range).

## The Workflow

1. **Reduce the report to a minimal trigger.** Strip the reported scenario to the smallest input that still misbehaves (delta-debugging by hand: halve the input, keep the failing half). The minimal trigger names the real bug — "crashes on orders with a deleted product" beats "crashes on Karen's order from Tuesday."

2. **Write the test at the lowest layer that exhibits the bug.**

   ```
   Reproduced at the reported layer (UI / API / job)
                     |
                     v
        Can one layer lower still exhibit it?
              |                   |
             yes                  no
              |                   |
              v                   v
        drop a layer,      write the test HERE
        re-check           (one layer above where
                            the bug disappears)
   ```

   If the broken logic is a pure function, unit test it there — don't write a browser test because the *report* came through the UI.

3. **Name it after the bug, link the ticket.**
   ```python
   def test_discount_not_applied_twice_on_retry():  # regression: #4412
   ```
   The link matters at 3am two years later when someone wants to delete the "weird" test.

4. **Run it and watch it fail — for the right reason.** The assert must fail with the *reported* wrong behavior, not an unrelated error in setup. Capture the failing output (paste into the PR); a regression test that fails for the wrong reason will pass for the wrong reason after any change.

5. **Now write the fix.** Test goes green. Nothing else in the suite goes red (if it does, either the fix is wrong or a test was frozen around the bug — decide which, explicitly).

6. **Sweep for siblings.** The bug had a cause (unchecked null, race, off-by-one). Grep for the same pattern elsewhere in the codebase; each hit gets the same treatment or a ticket. Bugs cluster — the reporter found one instance of a class.

7. **Ask what let it escape.** One sentence in the PR: which missing test category (edge case? integration seam? concurrency?) would have caught this class, and whether it's worth adding systematically. This is the step that improves the suite instead of just patching it.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "The fix is one line — a test is overkill" | One-line fixes regress exactly as often as big ones; the next refactor doesn't know the line was load-bearing. The test costs minutes and guards forever. |
| "I'll add the test after the fix" | A post-fix test is written against fixed code — it was never seen detecting the bug, so it detects nothing provable. Order is the entire method. |
| "It's hard to reproduce; I'll just fix the likely cause" | No repro = no proof the fix fixes anything. "Likely cause" patches close tickets and reopen incidents. Reproduction IS the diagnosis. |
| "The e2e suite will catch this class of thing" | E2E tests are slow, flaky, and first in line for quarantine — which un-guards the bug (see `e2e-test-triage`). The minimal-layer test survives suite triage. |
| "Deadline pressure — I'll file a follow-up ticket for the test" | Follow-up test tickets have a near-zero completion rate. The test is part of the fix's definition of done, not an accessory. |
| "The bug is in a third-party dependency, not our code" | Freeze the correct behavior at your boundary anyway — the next dependency upgrade can reintroduce it, and your test is the only tripwire you control. |

## Red Flags

Signs this skill is being skipped or violated:

- The fix commit/PR contains no test change.
- A test exists but nobody ever saw it fail — no failing run recorded before the fix landed.
- The test asserts the fix's implementation ("retry counter == 1") instead of the reported behavior ("charged exactly once").
- The repro is a five-service scenario for what is a pure-function bug — wrong layer, will be quarantined within a year.
- Ticket closed with "fixed, could not reproduce."
- The same-pattern grep was never run — one instance fixed, siblings still live.

## Verification

Done means all boxes checked, each with evidence:

- [ ] Test fails on pre-fix code, with the *reported* wrong behavior — failing output captured in the PR.
- [ ] Test passes post-fix; full suite green — CI run linked.
- [ ] Test name states the behavior; ticket/issue linked in a comment.
- [ ] Sibling sweep run — the grep pattern and its hits (fixed or ticketed) listed in the PR.
- [ ] One-sentence "what let this escape" answer in the PR description.

## Example

Report: "customer charged twice." Step 1 minimized to: retry after timeout re-runs charge because the idempotency key was regenerated per attempt. Step 2: bug exhibits at the `PaymentClient` layer — unit test with a fake gateway asserting one charge across a timeout-retry. Step 4: fails showing 2 charges (output pasted into PR). Fix: key derived from order ID, not per-call UUID. Step 6 grep for `uuid4()` near retry decorators found the same pattern in refunds — second ticket filed. Step 7: added a "retry-safety" section to the review checklist.

## Related skills

- `debugging-by-bisection` — when reproduction is the hard part.
- `unit-test-design` — the structure/naming rules the regression test follows.
- `edge-case-enumeration` — the proactive sibling; this skill is the reactive one.
- `blameless-postmortem` — step 7 at incident scale.
