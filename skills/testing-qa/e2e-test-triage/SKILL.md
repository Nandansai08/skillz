---
name: e2e-test-triage
description: >
  Use when an end-to-end/browser test suite is slow, flaky, or bloated and
  needs triage — deciding what deserves e2e coverage, demoting the rest,
  stabilizing what stays. Triggers: "e2e tests are flaky", "Cypress/
  Playwright suite takes forever", "which tests should be e2e", "trim the
  e2e suite", "smoke tests". NOT for root-causing one specific flaky test
  (see flaky-test-diagnosis) — this skill decides the suite's shape.
---

# E2E Test Triage

## Overview

E2E suites grow by inertia until they block every merge with 40 minutes of 4%-flake theater. Triage classifies every test — journey-critical, logic-in-disguise, or inertia — keeps a small stabilized PR gate, and demotes the rest to layers that can actually hold them.

## When to Use

- The e2e suite is over budget (time or flake rate) and blocking merges.
- Designing which flows get e2e coverage in the first place.

**When NOT to use:**
- Diagnosing one flaky test's root cause — `flaky-test-diagnosis`.

## Prerequisites

- CI history: per-test duration and pass rate over the last few weeks (Playwright/Cypress dashboards, or CI job logs).

## The Workflow

1. **Set the budget first.** Working defaults: e2e suite ≤ 10 min wall-clock on PRs, flake rate ≤ 0.5% per run. Everything else follows from enforcing this; without a number, the suite only grows.

2. **Classify every existing e2e test into three bins:**
   - **Journey-critical:** the ~5–15 flows where breakage = revenue/trust loss (signup, login, checkout, core action). Keep at e2e.
   - **Logic-in-disguise:** tests driving a browser to verify a calculation, validation message, or permission rule. Demote to unit/integration — the browser adds cost, not confidence, for logic.
   - **Coverage-by-inertia:** old features, edge variants of kept journeys ("checkout with each of 9 payment methods"). Keep one representative path; demote or delete the variants.
   The usual finding: bin 1 is a fifth of the suite.

3. **Demote with a receiving test.** Each demotion names where the coverage lands (component test, API integration test) *in the same PR*. Deleting e2e tests without receivers is how regressions re-enter.

4. **Stabilize the keepers — the big four fixes:**
   - **No sleeps; only condition waits.** Every `wait(3000)` becomes wait-for-selector/response (`await expect(locator).toBeVisible()`).
   - **Own your test data.** Each test creates its user/org via API seed and gets isolated state; shared "test@test.com" accounts are cross-test interference by design.
   - **Stub the third parties.** Payment sandboxes, email, analytics — network-stub at the edge (`page.route`). Their downtime shouldn't fail your merge.
   - **Login via API, not UI.** Test the login *flow* once; every other test injects a session token.
   Warning: selectors on CSS classes/DOM structure fail on restyles — use `data-testid` or role+name.

5. **Split the suite by trigger:** PR gate runs the journey-critical set; the fuller set runs post-merge/nightly with alerting. A nightly failure is a ticket; a PR-gate failure blocks — reserve blocking for the tests that earn it.

6. **Enforce ongoing:** new e2e test requires a one-line justification for why it can't be a lower-layer test; quarantine with owner+deadline for any test exceeding the flake threshold; monthly duration/pass-rate review against the step-1 budget. Keep screenshots/video on failure — when the 1-in-50 flake fires, the artifact IS the diagnosis.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "E2E is the most realistic — more of it means more confidence" | Realism per test, bankruptcy per suite: 200 browser tests at 4% flake means every merge rolls dice. Confidence comes from the RIGHT 20 e2e tests plus solid lower layers. |
| "Just set retries: 3 — problem solved" | Retries hide real races (sometimes app races), triple worst-case CI time, and remove all pressure to fix. Acceptable only WITH flake tracking that counts a retried pass as a flake. |
| "We can't delete tests — what if something regresses?" | That's why demotion requires a receiving test in the same PR (step 3). The choice isn't coverage vs none; it's coverage at a layer that can hold it vs coverage that gets quarantined. |
| "The discount-math e2e test catches real bugs" | At 40 seconds per run and browser-flake rates — the same bug fails a unit test in milliseconds, deterministically. The browser is testing the math's SCENERY. |
| "One long journey test covers everything efficiently" | First failure masks the rest, it can't parallelize, and its failure message is 'somewhere in 4 minutes of clicking.' Split by journey. |
| "Testing through the UI because the UI exists" | Existence isn't a test strategy. Each e2e test needs the step-6 justification: why can't a cheaper layer catch this? |

## Red Flags

- Merge queue dominated by e2e reruns; devs muscle-memory the retry button.
- `wait(N)` sleeps scattered through specs.
- A shared test account whose state depends on which tests ran first.
- Suite runtime and test count only ever increasing; no monthly review.
- Real third-party sandboxes on the PR-gate path.
- E2E tests deleted with no receiving test named.

## Verification

- [ ] Budget stated (max minutes, max flake rate) and current numbers measured against it — dashboard linked.
- [ ] Classification table exists: every e2e test binned with disposition — artifact linked.
- [ ] Every demotion PR names its receiving test — spot-check three.
- [ ] Keepers pass the big-four audit (no sleeps, owned data, stubbed third parties, API login) — grep results for `wait(`/shared accounts attached.
- [ ] PR-gate vs nightly split implemented — CI config linked.
- [ ] Flake tracking counts retried passes as flakes — config or dashboard shown.

## Example

Suite: 212 Playwright tests, 38 min, 4% flake, merges queued behind reruns. Classification: 19 journey-critical, 118 logic-in-disguise (mostly form-validation variants), 75 inertia. Result after two weeks: 24 e2e tests on PRs (7 min), validation logic moved to component tests (run in 40s), payment stubs + API login applied to keepers, nightly runs the extended 60-test set. Flake rate 0.3%; two months later a real checkout regression was caught by the PR gate in one run — visible precisely because red now means broken.

## Related skills

- `flaky-test-diagnosis` — root-causing individual keepers that still flake.
- `integration-test-strategy` — where the demoted tests land.
- `ci-pipeline-design` — wiring the PR-gate/nightly split.
