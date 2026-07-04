---
name: unit-test-design
description: >
  Use when writing unit tests and deciding what to test, how to structure
  and name them, and what to leave untested. Triggers: "write unit tests
  for", "test this function", "are these tests good", "how should I
  structure tests", "what should I test here". NOT for choosing mocks vs
  fakes (see test-doubles-choice) and NOT for integration-level scope
  decisions (see integration-test-strategy).
---

# Unit Test Design

## Overview

A unit test is one behavior, verified by outcome, named so its failure reads as a bug report. Tests written by any other recipe either break on every refactor or pass on real bugs — usually both.

## When to Use

- Writing tests for a new function/class, or reviewing a test file.
- Deciding whether something is worth a unit test at all.

**When NOT to use:**
- Choosing test doubles — `test-doubles-choice`.
- Deciding what belongs at integration scope — `integration-test-strategy`.

## Prerequisites

- The code under test, and its *intended* behavior — from a spec, ticket, or the author. Tests written by reading the implementation just freeze bugs.

## The Workflow

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

5. **Use builders/factories for arrange noise.** `make_coupon(expires=YESTERDAY)` with sensible defaults keeps each test showing only the detail that matters. Warning: share builders, never instances — shared mutable fixtures across tests are the ordering-dependent flakiness factory.

6. **Cover the boundaries the behavior implies.** For each behavior, the interesting inputs are: the boundary itself, one past it, empty/zero/null, and the type's weird values (see `edge-case-enumeration`). Parameterize when the cases share a shape:
   ```python
   @pytest.mark.parametrize("total,expected", [(0, 0), (0.01, 0.01), (100, 95)])
   ```

7. **Decide what NOT to test.** Skip: trivial pass-throughs, framework behavior (the ORM saves things), private helpers (test through the public surface), and generated code. Every skipped category is a maintenance cost you didn't buy.

8. **Red-check each test.** Break the code (or write test-first) and confirm the test fails *with a readable message*. A test that never went red is unverified; a cryptic failure message is a 3am tax. Warning: never use real time or sleeps — inject the clock (`freezegun`, fake timers); real-time tests are flaky tests on a delay.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "It runs without error — good enough as a test" | `test_runs_without_error` verifies the absence of exceptions, not the presence of correctness. Coverage without assertions is execution theater. |
| "One big test covering the whole flow is more efficient" | The 20-assert mega-test hides 19 failures behind the first, and its name can't say what broke. Efficiency in writing, bankruptcy in debugging. |
| "Asserting the mock was called proves it works" | If deleting the implementation body wouldn't fail the test, the test verifies stubs talking to each other. Outcomes or it's décor. |
| "I don't need to see it fail — it's obviously testing the right thing" | Tests that never went red have a long history of asserting nothing (wrong fixture, wrong path, tautological assert). The red-check is 30 seconds. |
| "Testing private helpers directly is more thorough" | It welds the suite to the implementation; every refactor breaks tests that guarded nothing user-visible. The public surface is the contract. |
| "We need 100% coverage, so test the getters too" | Coverage targets met with valueless tests hide the branch that matters at 91%. Test behaviors; let coverage report, not govern (see coverage-analysis). |

## Red Flags

- Test names with numeric suffixes (`test_calc_2`) — behaviors unnamed, failures unreadable.
- Arrange sections longer than act+assert combined, repeated across tests.
- Asserts on internal call sequences for logic with checkable outcomes.
- `sleep()` anywhere in a unit test.
- A test file that never fails when the implementation is deliberately broken.
- Shared module-level fixtures mutated by tests.

## Verification

- [ ] Behavior list exists (even as test names) — every test maps to one behavior.
- [ ] Each new test was seen red with a readable failure message — noted in PR or demonstrated by TDD commits.
- [ ] Assertions target outcomes; any interaction-assert justified as contract in a comment.
- [ ] Boundary cases from the behavior's edges present (or explicitly skipped with reason).
- [ ] No real time, no sleeps, no shared mutable fixtures — grep the diff for `sleep|now()|utcnow` and check.

## Example

Function: `split_bill(total, people, tip_pct)`. Behavior list produced 6 tests: even split exact; uneven split distributes remainder cents deterministically (largest-remainder, first payers); zero people raises `ValueError`; tip rounds half-even; negative total raises; single person pays all. The remainder-cents test caught a real bug — naive `total/people` rounding lost a cent on 100/3. Coverage of the file: 91%; the untested 9% is a `__repr__`, deliberately skipped.

## Related skills

- `test-doubles-choice` — when a dependency needs replacing to isolate the unit.
- `edge-case-enumeration` — a systematic source for step 6's inputs.
- `regression-test-from-bug` — the special case where the behavior list comes from a bug report.
