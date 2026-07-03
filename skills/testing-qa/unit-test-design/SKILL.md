---
name: unit-test-design
description: >
  Use when writing unit tests and deciding what to test, how to structure and
  name them, and what to leave untested. Triggers: "write unit tests for",
  "test this function", "are these tests good", "how should I structure
  tests", "what should I test here".
---

# Unit Test Design

## When to use this skill
- Writing tests for a new function/class, or reviewing a test file.
- Deciding whether something is worth a unit test at all.
- NOT for choosing mocks vs fakes (`test-doubles-choice`) or integration-level scope (`integration-test-strategy`).

## Prerequisites
- The code under test, and its *intended* behavior — from a spec, ticket, or the author. Tests written by reading the implementation just freeze bugs.

## Workflow

1. **List behaviors, not methods.** For the unit under test, enumerate observable behaviors: "rejects expired coupons", "rounds to cents half-even", "returns empty list for unknown user". One test per behavior. A method with five behaviors gets five tests; a trivial getter gets zero.

2. **Structure every test as Arrange-Act-Assert**, visually separated:
   ```python
   def test_expired_coupon_is_rejected():
       coupon = make_coupon(expires=YESTERDAY)          # arrange
       result = apply_coupon(cart_total=100, coupon=coupon)  # act
       assert result.rejected_reason == "expired"        # assert
   ```
   One act per test. Multiple asserts are fine when they verify one behavior (status + reason); asserting two behaviors means two tests.

3. **Name tests as behavior statements** that read as documentation when they fail: `test_expired_coupon_is_rejected`, not `test_apply_coupon_2`. The failure list should describe the broken contract without opening the file.

4. **Assert on outcomes, not implementation.** Return values, state changes, emitted events — not "method X was called with Y" unless the call *is* the contract (e.g., "sends exactly one email"). Implementation-coupled tests fail on every refactor and pass on real bugs.

5. **Use builders/factories for arrange noise.** `make_coupon(expires=YESTERDAY)` with sensible defaults keeps each test showing only the detail that matters. Ten lines of irrelevant setup per test is how test files become unreadable.

6. **Cover the boundaries the behavior implies.** For each behavior, the interesting inputs are: the boundary itself, one past it, empty/zero/null, and the type's weird values (see `edge-case-enumeration`). Parameterize when the cases share a shape:
   ```python
   @pytest.mark.parametrize("total,expected", [(0, 0), (0.01, 0.01), (100, 95)])
   ```

7. **Decide what NOT to test.** Skip: trivial pass-throughs, framework behavior (the ORM saves things), private helpers (test through the public surface), and generated code. Every skipped category is a maintenance cost you didn't buy.

8. **Red-check each test.** Break the code (or write test-first) and confirm the test fails *with a readable message*. A test that never went red is unverified; a cryptic failure message is a 3am tax.

## Common pitfalls
- Testing the mock: so much is stubbed that the test verifies the stubs talk to each other. If deleting the implementation body wouldn't fail the test, it tests nothing.
- Shared mutable fixtures across tests — the ordering-dependent flakiness factory. Fresh state per test; share builders, not instances.
- One mega-test walking a whole scenario with 20 asserts. First failure hides the other 19; split by behavior.
- Chasing 100% coverage by asserting nothing (`test_runs_without_error`). Coverage measures execution, not verification.
- Sleeping or using real time. Inject the clock; `freezegun`/fake timers. Real-time tests are flaky tests on a delay.

## Example
Function: `split_bill(total, people, tip_pct)`. Behavior list produced 6 tests: even split exact; uneven split distributes remainder cents deterministically (largest-remainder, first payers); zero people raises `ValueError`; tip rounds half-even; negative total raises; single person pays all. The remainder-cents test caught a real bug — naive `total/people` rounding lost a cent on 100/3. Coverage of the file: 91%; the untested 9% is a `__repr__`, deliberately skipped.

## Related skills
- `test-doubles-choice` — when a dependency needs replacing to isolate the unit.
- `edge-case-enumeration` — a systematic source for step 6's inputs.
- `regression-test-from-bug` — the special case where the behavior list comes from a bug report.
