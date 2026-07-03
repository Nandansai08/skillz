---
name: e2e-test-triage
description: >
  Use when an end-to-end/browser test suite is slow, flaky, or bloated and
  needs triage — deciding what deserves e2e coverage, demoting the rest,
  stabilizing what stays. Triggers: "e2e tests are flaky", "Cypress/Playwright
  suite takes forever", "which tests should be e2e", "trim the e2e suite",
  "smoke tests".
---

# E2E Test Triage

## When to use this skill
- The e2e suite is over budget (time or flake rate) and blocking merges.
- Designing which flows get e2e coverage in the first place.
- NOT for diagnosing one specific flaky test's root cause — that's `flaky-test-diagnosis`; this skill decides the suite's shape.

## Prerequisites
- CI history: per-test duration and pass rate over the last few weeks (Playwright/Cypress dashboards, or CI job logs).

## Workflow

1. **Set the budget first.** Working defaults: e2e suite ≤ 10 min wall-clock on PRs, flake rate ≤ 0.5% per run. Everything else follows from enforcing this; without a number, the suite only grows.

2. **Classify every existing e2e test into three bins:**
   - **Journey-critical:** the ~5–15 flows where breakage = revenue/trust loss (signup, login, checkout, core action). Keep at e2e.
   - **Logic-in-disguise:** tests driving a browser to verify a calculation, validation message, or permission rule. Demote to unit/integration — the browser adds cost, not confidence, for logic.
   - **Coverage-by-inertia:** old features, edge variants of kept journeys ("checkout with each of 9 payment methods"). Keep one representative path; demote or delete the variants.
   The usual finding: bin 1 is a fifth of the suite.

3. **Demote with a receiving test.** Each demotion names where the coverage lands (component test, API integration test) *in the same PR*. Deleting e2e tests without receivers is how regressions re-enter.

4. **Stabilize the keepers — the big four fixes:**
   - **No sleeps; only condition waits.** Every `wait(3000)` becomes wait-for-selector/response (`await expect(locator).toBeVisible()`; auto-waiting assertions).
   - **Own your test data.** Each test creates its user/org via API seed and gets isolated state; shared "test@test.com" accounts are cross-test interference by design.
   - **Stub the third parties.** Payment sandboxes, email, analytics — network-stub at the edge (`page.route`). Their downtime shouldn't fail your merge.
   - **Login via API, not UI.** Test the login *flow* once; every other test injects a session token. Cuts minutes and removes the most-repeated flake source.

5. **Split the suite by trigger:** PR gate runs the journey-critical set; the fuller set runs post-merge/nightly with alerting. A nightly failure is a ticket; a PR-gate failure blocks — reserve blocking for the tests that earn it.

6. **Enforce ongoing:** new e2e test requires a one-line justification for why it can't be a lower-layer test; quarantine with owner+deadline for any test exceeding the flake threshold; monthly duration/pass-rate review against the step-1 budget.

## Common pitfalls
- Retries as policy (`retries: 3`). It hides real races (sometimes app races, not test races), triples worst-case CI time, and removes the pressure to fix. Retries acceptable only *with* flake-rate tracking that still counts a retried pass as a flake.
- Testing through the UI because the UI exists. "Discount math is wrong" should fail a unit test in milliseconds, not a browser test in 40 seconds.
- One mega-journey test covering everything sequentially — first failure masks the rest, and it can't parallelize.
- Screenshots/video off to save time. When a 1-in-50 flake fires, the artifact *is* the diagnosis; keep them on failure-only.
- Selectors on CSS classes/DOM structure. Use `data-testid` or role+name; restyling shouldn't fail the suite.

## Example
Suite: 212 Playwright tests, 38 min, 4% flake, merges queued behind reruns. Classification: 19 journey-critical, 118 logic-in-disguise (mostly form-validation variants), 75 inertia. Result after two weeks: 24 e2e tests on PRs (7 min), validation logic moved to component tests (run in 40s), payment stubs + API login applied to keepers, nightly runs the extended 60-test set. Flake rate 0.3%; two months later a real checkout regression was caught by the PR gate in one run — visible precisely because red now means broken.

## Related skills
- `flaky-test-diagnosis` — root-causing individual keepers that still flake.
- `integration-test-strategy` — where the demoted tests land.
- `ci-pipeline-design` — wiring the PR-gate/nightly split.
