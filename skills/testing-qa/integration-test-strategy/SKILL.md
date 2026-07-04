---
name: integration-test-strategy
description: >
  Use when deciding what to test with real components wired together versus
  unit tests — scoping integration tests, test containers, contract tests,
  choosing seams. Triggers: "integration tests for", "should this be unit
  or integration", "test against a real database", "testcontainers", "how
  much integration testing". NOT for browser-driven full-system tests
  (see e2e-test-triage) and NOT for double-type selection (see
  test-doubles-choice).
---

# Integration Test Strategy

## Overview

Mocks encode your beliefs about a dependency; integration tests go exactly where those beliefs are least trustworthy — SQL actually executing, serialization round-tripping, framework wiring. One real integration per test, real engines not lookalikes, and the seams stop lying.

## When to Use

- Planning the test approach for a service or feature that spans components.
- A suite is all-units-all-mocked and production keeps breaking at the seams.

**When NOT to use:**
- Browser-driven full-system flows — `e2e-test-triage`.
- Choosing what to stub on the non-tested side — `test-doubles-choice`.

## Prerequisites

- Docker (or equivalent) available in dev and CI for real dependencies.
- Agreement on what "unit" means locally — this skill assumes units are fast, isolated, logic-focused.

## The Workflow

1. **Put integration tests where the lies are.** Integration tests go where beliefs are least trustworthy: SQL actually executing (syntax, constraints, transactions), ORM mappings, queue serialization round-trips, HTTP client + real serializer against a contract, framework wiring (DI, middleware order, auth filters).

2. **Choose the seam: test one real integration per test.** Service + real Postgres with the *outbound* HTTP stubbed is a good integration test. Service + real DB + real downstream + real queue is a small e2e — slow, and failures point nowhere. One lie checked at a time.

3. **Use real engines, not lookalikes.** Testcontainers (Postgres, Redis, Kafka, LocalStack) over in-memory substitutes: H2-instead-of-Postgres and fakeredis pass tests that fail in production on dialect, locking, and expiry semantics. Pin the container tag to the production version:
   ```python
   @pytest.fixture(scope="session")
   def pg():
       with PostgresContainer("postgres:16.3") as pg: yield pg
   ```

4. **Make isolation structural, not disciplined.** Per-test transaction-rollback (fastest), per-worker schema/database (for parallel runs), or truncate-between-tests (slowest, most compatible). Never share mutable rows across tests and hope cleanup runs — that's the flaky-suite starter kit.

5. **Migrate, don't create.** Build the test schema by running the real migrations, not `create_all()` from models. Otherwise migrations rot untested until a deploy fails.

6. **For cross-service boundaries, prefer contract tests over spinning up the other service.** Consumer writes expectations (Pact or recorded fixtures verified against the provider in *their* CI); each side tests against the contract independently. Booting service B inside service A's CI couples deploys and doubles flake surface.

7. **Budget the pyramid by feedback time.** Working default: units < 1 min for the repo, integration < 5 min, run per-PR. If integration exceeds the budget: parallelize by schema, session-scope the containers, and demote tests that check pure logic through the DB back into units — DB round-trips to test an `if` statement is the common bloat.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "SQLite in memory is faster and basically the same" | JSON operators, `RETURNING`, isolation levels, case-sensitivity — the deltas between engines are exactly where the bugs live. Fast tests of the wrong database verify the wrong database. |
| "Our mocks are carefully written to match the real API" | Mocks encode beliefs at writing time; the dependency evolves. The integration test is the only thing that re-checks the belief automatically. |
| "Docker in CI is slow, we run integration tests locally" | 'Locally' means 'on the one laptop that remembers to' — i.e., nowhere. Session-scoped containers + parallelized schemas fit the 5-min budget; the example below paid +3m10s. |
| "create_all() from models is equivalent to running migrations" | Until the migration that diverges from the models — which then rots untested until a production deploy discovers it. |
| "More real components per test = more realistic = better" | Multi-seam tests are small e2es: slow, flaky, and their failures point at nothing. Realism per seam, one seam per test. |
| "We test the other team's service by booting it in our CI" | Now your merges depend on their build health, and both flake surfaces multiply. Contract tests decouple the teams while keeping the guarantee. |

## Red Flags

- Production incidents recurring at seams (constraint violations, serialization mismatches) while the suite stays green.
- H2/SQLite/fakeredis standing in for a different production engine.
- Integration tests asserting on internal method calls — paid for the container, still testing implementation.
- Tests relying on rows a previous test inserted (fails on `-k`, unparallelizable).
- Test schema built by `create_all()`; migration files with zero test executions.
- Every logic branch driven through the DB — integration bloat carrying unit work.

## Verification

- [ ] Seam map exists: which lies get integration coverage, one seam per test — visible in the test plan or directory structure.
- [ ] Engines match production (image tags pinned to prod versions) — fixture code linked.
- [ ] Isolation structural: rollback/schema-per-worker/truncate — no inter-test data dependencies (suite passes with random order + parallel).
- [ ] Schema built via real migrations — the migration-run visible in test setup.
- [ ] Suite fits budget on PRs (time linked from CI); logic-through-DB tests demoted.
- [ ] Cross-service boundaries covered by contract tests, both sides — CI links.

## Example

Payments service: suite was 400 units, everything mocked; three consecutive incidents were SQL constraint violations and a Kafka serialization mismatch — all seams. Added: 25 repository tests against `postgres:16.3` via testcontainers with rollback isolation (checks real SQL + migrations), 6 round-trip tests on Kafka serialization against `redpanda`, and Pact contracts with the two consumer teams. CI cost: +3m10s. Next quarter's seam incidents: zero; two migration bugs caught pre-merge by step 5.

## Related skills

- `test-doubles-choice` — what to stub on the non-tested side of the seam.
- `e2e-test-triage` — the layer above; keep it thin because this layer is solid.
- `data-pipeline-idempotency` — the analogous seam-testing mindset for pipelines.
