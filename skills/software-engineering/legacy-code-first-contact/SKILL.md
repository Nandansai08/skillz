---
name: legacy-code-first-contact
description: >
  Use when you must change unfamiliar, untested, or poorly documented code
  and can't verify what it currently does. Builds a characterization-test
  safety net before touching anything. Triggers: "no tests on this code",
  "afraid to touch this", "inherited this codebase", "what does this even
  do", "modify legacy code".
---

# Legacy Code First Contact

## When to use this skill
- Changing code with no tests, no docs, and no original author available.
- Onboarding onto a system where the risk of breaking hidden behavior is high.
- NOT for greenfield code or code with a healthy suite — just change it and run the tests.

## Prerequisites
- Ability to run the code locally or in a sandbox (or, failing that, access to production logs showing real inputs/outputs).

## Workflow

1. **Find the seams before reading everything.** You don't need to understand the whole system — only the region your change touches. Trace inward from an entry point (route handler, CLI arg, cron job) to your target, listing every function on the path.

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

6. **Now make the change**, running the characterization suite after each step. A failing characterization test means you changed behavior — decide *consciously* whether that's the intended change or a break.

7. **Leave the net behind.** Promote characterization tests that reflect intended behavior into the real suite; delete ones frozen around bugs you fixed, replacing each with a test asserting the correct behavior.

## Common pitfalls
- "Fixing" weird behavior found in step 2. Some downstream consumer probably depends on it (Hyrum's Law). Log it, freeze it, fix it as a separate, announced change.
- Characterizing through too many layers — asserting on final HTML when you're changing a fee function buries the signal. Test at the closest seam.
- Refactoring to "make it testable" before tests exist. That's the trap; use the two minimal dependency-breaks in step 4 instead.
- Trusting names and comments. In legacy code, `validate_user` may send email. Verify by reading the body or tracing a call, never by signature.

## Example
Task: change rounding in a 12-year-old billing module, zero tests. Traced the seam: `generate_invoice → apply_discounts → round_total`. Pulled 50 real invoices from logs, wrote 50 characterization asserts on `apply_discounts` + `round_total`. Two surprises frozen: negative discounts allowed, and totals round *down* for one legacy customer tier. Made the rounding change; 3 characterization tests failed — exactly the 3 invoices the change was supposed to affect. Other 47 green: no collateral damage. Shipped with the suite left in place.

## Related skills
- `refactor-safely` — the follow-on once the safety net exists.
- `regression-test-from-bug` — freezing a specific reported bug instead of general behavior.
- `monorepo-navigation` — finding the entry points and seams faster in a large repo.
