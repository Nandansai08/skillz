---
name: deployment-rollback-plan
description: >
  Use when designing deploys that can be undone — rollback procedures,
  migration-safe releases, expand/contract schema changes, feature-flag
  kill switches. Triggers: "rollback plan", "can we roll this back",
  "deploy with a migration", "backward compatible deploy",
  "safe database migration", "blue-green vs canary".
---

# Deployment Rollback Plan

## When to use this skill
- Planning a release that includes schema changes, API changes, or anything stateful.
- Writing the rollback section of a deploy checklist or runbook.
- NOT for executing an in-progress incident rollback — that's `incident-triage`; this skill is what makes that rollback possible.

## Prerequisites
- Knowledge of what the deploy changes: code only, schema, data backfill, config, external contracts.

## Workflow

1. **Classify the deploy's rollback difficulty upfront:**
   - **Code-only:** trivially rollbackable — redeploy previous artifact. This is the state to engineer toward.
   - **Schema change:** rollbackable only if expand/contract discipline is followed (step 3).
   - **Data migration/backfill, external side effects (emails sent, webhooks fired), contract changes consumed by others:** rollback ≠ undo. These need forward-fix plans, not rollback plans — say so explicitly.

2. **Enforce N/N-1 compatibility.** During any rollout, old and new code run simultaneously against the same DB and queues; after a rollback, old code runs against the new schema. Rule: every deploy must work with the schema/messages of the deploy before and after it. This single rule generates most of the rest.

3. **Schema changes: expand → migrate → contract, as separate deploys.**
   - **Expand:** add the new nullable column/table/index. Old code ignores it. Fully rollback-safe.
   - **Migrate:** new code writes both old+new (or reads new, falls back old); backfill runs online, batched, resumable.
   - **Contract:** only after the previous deploy has soaked and no reader of the old column remains, drop it — in its own deploy, weeks later is fine.
   Never `DROP`/`RENAME`/type-change in the same deploy as the code that stops using it. Renames are add-new + contract-old, never `ALTER ... RENAME`.

4. **Decouple risky behavior from the deploy with flags.** Ship dark, enable gradually, and get a kill switch that's faster than any redeploy (seconds vs minutes). The flag *is* the rollback for behavior changes; the deploy rollback then only covers crashes.

5. **Write the rollback procedure before deploying, and make it executable:** the exact command (`helm rollback api 42` / redeploy pipeline for artifact `v1.41.2`), who can run it, expected duration, and the **rollback triggers** — objective lines like "5xx rate > 1% for 5 min" or "checkout conversion drops >10%" decided now, because during the incident is when judgment is worst.

6. **Match rollout shape to blast radius.** Canary (1% → 10% → 100% with automated metric gates) as default for services; blue-green when you need instant full cutback and can afford double capacity; rolling for the boring middle. All three assume step 2's compatibility — none of them save you from a breaking migration.

7. **Test the rollback path.** In staging: deploy N, migrate, deploy N+1, then actually roll back to N and run the smoke suite against the expanded schema. An untested rollback plan is a hypothesis. Do the same for the kill switch.

## Common pitfalls
- "We'll write a down-migration." Down-migrations that drop columns destroy the data written since deploy; on real incidents nobody runs them. Design so rollback needs *no* schema reversal (expand/contract does this).
- Deleting a column the same week code stopped reading it — a rollback now brings back code that reads a dropped column. Soak the contract phase.
- Queue/message changes forgotten: new producer emits a field old consumers crash on. N/N-1 applies to every contract, not just SQL.
- Rollback plan says "revert the commit" for a change that sent 40k emails. Classify honestly (step 1); irreversible effects need flags + gradual rollout, not revert theater.
- Kill switch never exercised — flag flips off, code path was never tested dark, outage anyway. Step 7 covers flags too.

## Example
Deploy: replace `orders.status` string with a state-machine enum + new `state` column. Plan produced: Deploy 1 adds nullable `state`, dual-write behind flag `orders_state_v2` (off). Deploy 2: flag to 5% canary with gate "state/status mismatch metric == 0", backfill job (batched 10k rows, resumable, ran 6h). Deploy 3, next week: reads switch to `state`, `status` still written. Deploy 4, three weeks later: contract — stop writing `status`; column dropped a month after. During Deploy 2 the mismatch metric fired (legacy cron wrote `status` directly, bypassing the ORM) — flag off in 20 seconds, cron fixed, re-canaried next day. No rollback ever touched the schema.

## Related skills
- `incident-triage` — the consumer of step 5's triggers and procedures.
- `environment-parity` — staging that can actually rehearse step 7's tests.
- `error-handling-strategy` — the mismatch-metric pattern is its "loud fallback" rule.
