---
name: test-doubles-choice
description: >
  Use when a test needs to replace a real dependency and you must choose
  between mock, stub, fake, or spy — or when a test suite is drowning in
  mocks. Triggers: "should I mock this", "mock vs stub", "too many mocks",
  "fake for the database", "how to test code that calls an API".
---

# Test Doubles Choice

## When to use this skill
- A unit under test depends on something slow, nondeterministic, or external.
- Reviewing a suite where every test starts with ten `mock.patch` lines.
- NOT for deciding test scope — if the answer is "don't replace it, test through it," that's `integration-test-strategy`.

## Prerequisites
- Clarity on what the test is verifying: state (what the code returns/changes) or interaction (what the code tells a collaborator to do). The double type follows from this.

## Workflow

1. **First ask: does this need a double at all?** Pure logic extracted from the I/O call often removes the need — test the logic directly, leave one thin integration test for the wiring. The best double is no double.

2. **Pick by what you're verifying:**
   - **Stub** — returns canned data; test verifies *state*. Default choice. "When the rate API returns 1.1, the total is 110."
   - **Fake** — working lightweight implementation (in-memory repo, local filesystem, SQLite for Postgres). Use when many tests need realistic behavior (querying, uniqueness) — one fake replaces fifty per-test stub setups.
   - **Spy** — records calls; test asserts on them *after acting*. Use when the interaction is the contract: "publishes exactly one OrderShipped event."
   - **Mock (strict, expectation-first)** — fails on unexpected calls. Rarely the right default; use only when *not* calling something is the requirement ("does NOT charge the card twice").

3. **Replace at the boundary you own.** Double your own `PaymentGateway` interface, not the vendor SDK's 40-method client. Wrap third-party clients in a thin adapter and double the adapter — vendor API churn then breaks one adapter test, not 200 mocks.

4. **Keep fakes honest with contract tests.** Run the same test suite against the fake and the real implementation (in CI nightly if slow). An unverified fake drifts from reality and your green suite verifies fiction:
   ```python
   class RepoContract:            # shared behavior spec
       def test_get_after_save(self): ...
   class TestFakeRepo(RepoContract): repo = InMemoryRepo()
   class TestPgRepo(RepoContract): repo = PgRepo(test_db)   # marked slow
   ```

5. **Stub responses must be realistic.** Copy real payloads (sanitized) rather than hand-writing minimal ones — hand-written stubs omit the null field or extra key that breaks production parsing.

6. **Audit for overmocking.** Smells that mean the boundary is wrong: mocks returning mocks; `patch` targeting private functions of the module under test; a test that breaks when you rename an internal method; setup longer than assert+act combined. Fix by extracting logic (step 1) or introducing the adapter (step 3), not by adding more patches.

## Common pitfalls
- Asserting interactions when state was available. "Called save() with X" breaks on refactor to batch-save; "the record exists after" survives it.
- Patching where it's defined instead of where it's used (`mock.patch("requests.get")` when the module did `from requests import get`). The classic Python no-op mock.
- Strict mocks everywhere: every internal call sequence becomes API, and refactors fail 80 tests that verify nothing user-visible.
- Faking what you don't understand — an in-memory "Postgres" without transaction semantics passes tests that deadlock in production. Contract tests (step 4) are the guardrail.

## Example
Suite for `OrderService` had 30 tests each patching 6 methods of the Stripe SDK; renaming an internal helper failed 22 of them. Fix: introduced a 3-method `PaymentGateway` adapter owned by the team. Tests now use a `FakeGateway` (records charges, can be told to decline); two contract tests run against real Stripe test-mode nightly. The double-charge protection test uses a strict mock — the one place "must not call" is the actual requirement. Rename refactors now break zero tests.

## Related skills
- `unit-test-design` — the tests these doubles slot into.
- `integration-test-strategy` — when to stop doubling and hit the real thing.
- `flaky-test-diagnosis` — doubles that hide nondeterminism vs fix it.
