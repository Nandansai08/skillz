---
name: test-doubles-choice
description: >
  Use when a test needs to replace a real dependency and you must choose
  between mock, stub, fake, or spy — or when a suite is drowning in mocks.
  Triggers: "should I mock this", "mock vs stub", "too many mocks", "fake
  for the database", "how to test code that calls an API". NOT for
  deciding test scope — if the answer is "don't replace it, test through
  it," that's integration-test-strategy.
---

# Test Doubles Choice

## Overview

The double type follows from what the test verifies: state wants stubs and fakes, interactions want spies, must-not-happen wants strict mocks. Choosing by habit instead — mock everything — produces suites that break on every rename and catch nothing real.

## When to Use

- A unit under test depends on something slow, nondeterministic, or external.
- Reviewing a suite where every test starts with ten `mock.patch` lines.

**When NOT to use:**
- When the right move is testing through the real dependency — `integration-test-strategy`.

## Prerequisites

- Clarity on what the test is verifying: state (what the code returns/changes) or interaction (what the code tells a collaborator to do). The double type follows from this.

## The Workflow

1. **First ask: does this need a double at all?** Pure logic extracted from the I/O call often removes the need — test the logic directly, leave one thin integration test for the wiring. The best double is no double.

2. **Pick by what you're verifying:**
   - **Stub** — returns canned data; test verifies *state*. Default choice. "When the rate API returns 1.1, the total is 110."
   - **Fake** — working lightweight implementation (in-memory repo, local filesystem, SQLite for Postgres). Use when many tests need realistic behavior (querying, uniqueness) — one fake replaces fifty per-test stub setups.
   - **Spy** — records calls; test asserts on them *after acting*. Use when the interaction is the contract: "publishes exactly one OrderShipped event."
   - **Mock (strict, expectation-first)** — fails on unexpected calls. Rarely the right default; use only when *not* calling something is the requirement ("does NOT charge the card twice").

3. **Replace at the boundary you own.** Double your own `PaymentGateway` interface, not the vendor SDK's 40-method client. Wrap third-party clients in a thin adapter and double the adapter — vendor API churn then breaks one adapter test, not 200 mocks. Warning (Python): patch where the name is *used*, not where it's defined — `mock.patch("requests.get")` is a no-op when the module did `from requests import get`.

4. **Keep fakes honest with contract tests.** Run the same test suite against the fake and the real implementation (in CI nightly if slow). An unverified fake drifts from reality and your green suite verifies fiction:
   ```python
   class RepoContract:            # shared behavior spec
       def test_get_after_save(self): ...
   class TestFakeRepo(RepoContract): repo = InMemoryRepo()
   class TestPgRepo(RepoContract): repo = PgRepo(test_db)   # marked slow
   ```

5. **Stub responses must be realistic.** Copy real payloads (sanitized) rather than hand-writing minimal ones — hand-written stubs omit the null field or extra key that breaks production parsing.

6. **Audit for overmocking.** Smells that mean the boundary is wrong: mocks returning mocks; `patch` targeting private functions of the module under test; a test that breaks when you rename an internal method; setup longer than assert+act combined. Fix by extracting logic (step 1) or introducing the adapter (step 3), not by adding more patches.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "Mock everything — that's what makes it a unit test" | Unit means fast and isolated, not stub-buried. A test whose setup is ten patches verifies the patches' choreography, not the code. |
| "Asserting save() was called with X is the same as checking state" | It welds the test to today's implementation: the refactor to batch-save breaks it while the actual data-persists contract holds. State assertions survive refactors; call assertions survive nothing. |
| "Our in-memory fake is close enough to Postgres" | Close-enough drifts: transaction semantics, uniqueness, case-sensitivity. Without contract tests the fake is a fiction generator with green output. |
| "Hand-written minimal stubs keep tests readable" | The minimal stub omits the null field production sends, and the parser bug ships. Sanitized real payloads cost one copy-paste. |
| "Strict mocks everywhere catch unexpected behavior" | They convert every internal call sequence into API — 80 tests fail on a harmless refactor, and the team learns to update mocks mechanically, which catches nothing. |
| "Wrapping the vendor SDK in an adapter is an extra layer" | The adapter is 3 methods; the alternative is 200 tests mocking a 40-method client that the vendor reshuffles yearly. |

## Red Flags

- `mock.patch` lines outnumbering assert lines across a test file.
- Mocks returning mocks returning mocks.
- Tests failing en masse on an internal rename with zero behavior change.
- A fake with no contract test anywhere in CI.
- Stub payloads with placeholder-shaped values (`"string"`, `"foo"`) feeding parsing code.
- Patch targets pointing at definition sites in from-import codebases (silently patching nothing).

## Verification

- [ ] Each double's type matches its verification target (state→stub/fake, contract-interaction→spy, must-not→strict mock) — reviewable per test.
- [ ] Doubles replace owned boundaries; vendor SDKs wrapped — no direct vendor-client mocking in the diff.
- [ ] Every fake has a contract test running against the real implementation — CI job linked.
- [ ] Stub payloads sourced from real (sanitized) responses — origin noted in fixture comments.
- [ ] Rename-safety spot check: an internal rename in the tested module breaks zero tests in the diff.

## Example

Suite for `OrderService` had 30 tests each patching 6 methods of the Stripe SDK; renaming an internal helper failed 22 of them. Fix: introduced a 3-method `PaymentGateway` adapter owned by the team. Tests now use a `FakeGateway` (records charges, can be told to decline); two contract tests run against real Stripe test-mode nightly. The double-charge protection test uses a strict mock — the one place "must not call" is the actual requirement. Rename refactors now break zero tests.

## Related skills

- `unit-test-design` — the tests these doubles slot into.
- `integration-test-strategy` — when to stop doubling and hit the real thing.
- `flaky-test-diagnosis` — doubles that hide nondeterminism vs fix it.
