---
name: integration-test-strategy
description: >
  Use when deciding what to test with real components wired together versus
  unit tests — scoping integration tests, using test containers, choosing
  seams. Triggers: "integration tests for", "should this be unit or
  integration", "test against a real database", "testcontainers",
  "how much integration testing".
---

# Integration Test Strategy

## When to use this skill
- Planning the test approach for a service or feature that spans components.
- A suite is all-units-all-mocked and production keeps breaking at the seams.
- NOT for browser-driven full-system tests — that's `e2e-test-triage`.

## Prerequisites
- Docker (or equivalent) available in dev and CI for real dependencies.
- Agreement on what "unit" means locally — this skill assumes units are fast, isolated, logic-focused.

## Workflow

1. **Put integration tests where the lies are.** Mocks encode your *beliefs* about a dependency. Integration tests go exactly where beliefs are least trustworthy: SQL actually executing (syntax, constraints, transactions), ORM mappings, queue serialization round-trips, HTTP client + real serializer against a contract, framework wiring (DI, middleware order, auth filters).

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

## Common pitfalls
- The "integrated unit test": real DB but asserting on internal method calls. You paid for the container and still tested implementation.
- In-memory SQLite standing in for Postgres. JSON operators, `RETURNING`, isolation levels, case-sensitivity — the deltas are exactly where the bugs are.
- One test relying on rows a previous test inserted. Works in order, fails on `-k`, unparallelizable.
- Testing every logic branch through the integration layer. Branches are unit territory; integration tests check the seam once per shape of interaction.
- Skipping integration tests in CI "because Docker is slow there" — the only place they run becomes a developer's laptop, i.e., nowhere.

## Example
Payments service: suite was 400 units, everything mocked; three consecutive incidents were SQL constraint violations and a Kafka serialization mismatch — all seams. Added: 25 repository tests against `postgres:16.3` via testcontainers with rollback isolation (checks real SQL + migrations), 6 round-trip tests on Kafka serialization against `redpanda`, and Pact contracts with the two consumer teams. CI cost: +3m10s. Next quarter's seam incidents: zero; two migration bugs caught pre-merge by step 5.

## Related skills
- `test-doubles-choice` — what to stub on the non-tested side of the seam (step 2).
- `e2e-test-triage` — the layer above; keep it thin because this layer is solid.
- `data-pipeline-idempotency` — the analogous seam-testing mindset for pipelines.
