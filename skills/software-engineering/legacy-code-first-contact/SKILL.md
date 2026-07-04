---
name: legacy-code-first-contact
description: >
  Use when you must change unfamiliar, untested, or poorly documented code
  and can't verify what it currently does — builds a characterization-test
  safety net before touching anything. Triggers: "no tests on this code",
  "afraid to touch this", "inherited this codebase", "what does this even
  do", "modify legacy code". NOT for greenfield code or code with a
  healthy suite (just change it and run the tests), and NOT for freezing
  a specific reported bug (see regression-test-from-bug).
---

# Legacy Code First Contact

## Overview

Untested legacy code punishes change with hidden-behavior breakage. Characterization tests — asserting what the code DOES, not what it should do — build the safety net that makes modification an engineering act instead of a gamble.

## When to Use

- Changing code with no tests, no docs, and no original author available.
- Onboarding onto a system where the risk of breaking hidden behavior is high.

**When NOT to use:**
- Greenfield code or code with a healthy suite — just change it and run the tests.
- A specific reported bug needing a failing-first test — `regression-test-from-bug` (that skill freezes *correct* behavior; this one freezes *current* behavior).

## Prerequisites

- Ability to run the code locally or in a sandbox (or, failing that, access to production logs showing real inputs/outputs).

## The Workflow

1. **Find the seams before reading everything.** You don't need to understand the whole system — only the region your change touches. Trace inward from an entry point (route handler, CLI arg, cron job) to your target, listing every function on the path. Warning: don't trust names or comments in legacy code — `validate_user` may send email; verify by reading bodies or tracing calls.

2. **Write characterization tests: assert what the code DOES, not what it should do.** Feed the target real-ish inputs, capture actual outputs, freeze them as assertions:
   ```python
   def test_characterize_fee_calc():
       # Not asserting correctness — asserting current behavior
       assert calc_fee(amount=100, tier="gold") == 3.5
       assert calc_fee(amount=0, tier="gold") == 3.5   # ...surprising, but it's what it does
       assert calc_fee(amount=-5, tier=None) == 3.5    # frozen weirdness, now documented
   ```
   Surprising outputs are the payload — each one is hidden behavior your change could silently alter. Freeze them, comment them, don't "fix" them yet.

3. **Harvest inputs from reality, not imagination.** Production logs, recorded requests, DB samples. Legacy behavior lives in the inputs nobody would think to make up (empty strings, legacy enum values, dates from 2011).

4. **Break dependencies minimally to get under test.** The two lowest-risk moves: extract-and-override (subclass, override the method that hits the DB) and parameterize-constructor (pass the dependency in, default to the old one). Avoid big DI refactors before you have the net — that's refactoring untested code, the thing you're here to avoid.

5. **Check coverage of the region you'll change.** Run coverage on just your target files. Uncovered branches inside your blast radius = more characterization tests needed. Ignore repo-wide numbers.

6. **Now make the change**, running the characterization suite after each step. A failing characterization test means you changed behavior — decide *consciously* whether that's the intended change or a break. Warning: characterize at the closest seam, not through five layers — asserting on final HTML when you're changing a fee function buries the signal.

7. **Leave the net behind.** Promote characterization tests that reflect intended behavior into the real suite; delete ones frozen around bugs you fixed, replacing each with a test asserting the correct behavior.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "This weird behavior is obviously a bug — I'll fix it while I'm in here" | Some downstream consumer probably depends on it (Hyrum's Law). "Fixing" it unannounced is how a refactor becomes an outage. Freeze it, log it, fix it as a separate, communicated change. |
| "The change is tiny — characterizing first is overkill" | Tiny changes to unknown code have unknown blast radii by definition. The characterization suite for one function is 30 minutes; the silent behavior break is a week. |
| "I'll make it testable first with a proper DI refactor" | That's refactoring untested code — the exact risk you came to avoid. The two minimal dependency-breaks (step 4) get you under test without restructuring anything. |
| "I can tell what it does from reading it" | Legacy code's actual behavior diverges from its apparent behavior precisely at the inputs you won't imagine (step 3). Reading gives hypotheses; characterization gives facts. |
| "Made-up test inputs are fine — I know the domain" | The 2011 date formats, empty strings, and dead enum values in production logs are the behavior. Imagined inputs characterize your model of the system, not the system. |
| "The function names and comments explain the flow" | In legacy code, names are historical fiction. Verify by execution, never by signature. |

## Red Flags

- A change to untested legacy code shipped with no new tests of any kind.
- Characterization tests all pass on the first run with zero surprising assertions — real legacy always surprises; no surprises means the inputs were too tame.
- "Cleaned up" weird behavior inside a supposedly behavior-preserving change.
- Tests asserting through many layers (HTTP → HTML) for a deep internal change.
- A big testability refactor landing BEFORE any tests exist.
- Coverage of the changed region unknown; confidence sourced from "it looks right."

## Verification

- [ ] Seam trace documented: entry point → target path listed (in PR or notes).
- [ ] Characterization tests exist, sourced from real inputs — at least one frozen "surprising" behavior with a comment.
- [ ] Coverage on the target region checked; uncovered branches in the blast radius addressed or explicitly accepted — number attached.
- [ ] The intended change's characterization diff is exact: only the tests covering the intended behavior change failed, and each failure was consciously dispositioned (PR notes which).
- [ ] Net left behind: promoted tests in the real suite; bug-frozen ones replaced with correct-behavior tests — commit linked.

## Example

Task: change rounding in a 12-year-old billing module, zero tests. Traced the seam: `generate_invoice → apply_discounts → round_total`. Pulled 50 real invoices from logs, wrote 50 characterization asserts on `apply_discounts` + `round_total`. Two surprises frozen: negative discounts allowed, and totals round *down* for one legacy customer tier. Made the rounding change; 3 characterization tests failed — exactly the 3 invoices the change was supposed to affect. Other 47 green: no collateral damage. Shipped with the suite left in place.

## Related skills

- `refactor-safely` — the follow-on once the safety net exists.
- `regression-test-from-bug` — freezing a specific reported bug instead of general behavior.
- `monorepo-navigation` — finding the entry points and seams faster in a large repo.
