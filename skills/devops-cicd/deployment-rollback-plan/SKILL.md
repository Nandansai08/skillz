---
name: deployment-rollback-plan
description: >
  Use when designing deploys that can be undone — rollback procedures,
  migration-safe releases, expand/contract schema changes, feature-flag
  kill switches. Triggers: "rollback plan", "can we roll this back",
  "deploy with a migration", "backward compatible deploy", "safe database
  migration", "blue-green vs canary". NOT for executing an in-progress
  incident rollback (see incident-triage) — this skill is what makes that
  rollback possible.
---

# Deployment Rollback Plan

## Overview

Rollback is designed, not improvised: expand/contract migrations, N/N-1 compatibility, and pre-written triggers decide at design time whether the 2am incident takes three minutes or three hours. Deploys that can't be undone need to be named as such before they ship.

## When to Use

- Planning a release that includes schema changes, API changes, or anything stateful.
- Writing the rollback section of a deploy checklist or runbook.

**When NOT to use:**
- Mid-incident execution — `incident-triage` runs the play; this skill wrote it.

## Prerequisites

- Knowledge of what the deploy changes: code only, schema, data backfill, config, external contracts.

## The Workflow

1. **Classify the deploy's rollback difficulty upfront:**
   - **Code-only:** trivially rollbackable — redeploy previous artifact. This is the state to engineer toward.
   - **Schema change:** rollbackable only if expand/contract discipline is followed (step 3).
   - **Data migration/backfill, external side effects (emails sent, webhooks fired), contract changes consumed by others:** rollback ≠ undo. These need forward-fix plans, not rollback plans — say so explicitly.

2. **Enforce N/N-1 compatibility.** During any rollout, old and new code run simultaneously against the same DB and queues; after a rollback, old code runs against the new schema. Rule: every deploy must work with the schema/messages of the deploy before and after it. This applies to every contract — queue messages included: a new producer field that old consumers crash on violates it as surely as a dropped column.

3. **Schema changes: expand → migrate → contract, as separate deploys.**
   - **Expand:** add the new nullable column/table/index. Old code ignores it. Fully rollback-safe.
   - **Migrate:** new code writes both old+new (or reads new, falls back old); backfill runs online, batched, resumable.
   - **Contract:** only after the previous deploy has soaked and no reader of the old column remains, drop it — in its own deploy, weeks later is fine.
   Never `DROP`/`RENAME`/type-change in the same deploy as the code that stops using it. Renames are add-new + contract-old, never `ALTER ... RENAME`.

4. **Decouple risky behavior from the deploy with flags.** Ship dark, enable gradually, and get a kill switch that's faster than any redeploy (seconds vs minutes). The flag *is* the rollback for behavior changes; the deploy rollback then only covers crashes.

5. **Write the rollback procedure before deploying, and make it executable:** the exact command (`helm rollback api 42` / redeploy pipeline for artifact `v1.41.2`), who can run it, expected duration, and the **rollback triggers** — objective lines like "5xx rate > 1% for 5 min" or "checkout conversion drops >10%" decided now, because during the incident is when judgment is worst.

6. **Match rollout shape to blast radius.** Canary (1% → 10% → 100% with automated metric gates) as default for services; blue-green when you need instant full cutback and can afford double capacity; rolling for the boring middle. All three assume step 2's compatibility — none of them save you from a breaking migration.

7. **Test the rollback path.** In staging: deploy N, migrate, deploy N+1, then actually roll back to N and run the smoke suite against the expanded schema. An untested rollback plan is a hypothesis. Do the same for the kill switch — a flag flipped off for the first time during an outage is a second experiment inside the first.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "We'll write a down-migration — that's our rollback" | Down-migrations that drop columns destroy the data written since deploy, and in real incidents nobody dares run them. Expand/contract makes rollback need NO schema reversal. |
| "The column's been unused since Tuesday, safe to drop with this release" | A rollback of this release brings back code that reads the dropped column — instant hard-down. Contract phases soak for weeks precisely because rollback windows outlive deploy windows. |
| "It's a small change, canary is overkill" | Small changes with big blast radii are the incident archive's most common entry. The canary costs minutes; its absence bets the fleet on 'small.' |
| "Revert the commit — that's the plan for the email campaign feature" | Reverting code doesn't unsend 40k emails. Irreversible-effect deploys need flags + gradual rollout; calling a revert a rollback here is theater (step 1's honest classification). |
| "We'll define rollback triggers if something looks wrong" | Mid-incident is where judgment is worst and sunk-cost is strongest. The trigger written calmly ('5xx > 1% for 5 min') executes itself; the vibes-based one debates itself. |
| "The kill switch is there; no need to test the dark path" | Untested flag-off paths have shipped broken plenty — outage anyway, now with extra confusion. Step 7 covers flags too. |

## Red Flags

- A single deploy containing both a schema drop/rename and the code change that stops using it.
- Rollback instructions that say "revert the commit" for stateful or side-effectful changes.
- No written rollback triggers; the deploy channel debating thresholds during an incident.
- Migration and feature riding one deploy "to save a release cycle."
- The staging environment has never once executed an actual rollback.
- Queue message schema changed with no consumer-compatibility note.

## Verification

- [ ] Deploy classified (code-only / schema / irreversible) in the deploy doc — with forward-fix plan where rollback ≠ undo.
- [ ] N/N-1 statement present: what old code does against new schema/messages, verified how.
- [ ] Schema changes split into expand/migrate/contract deploys — PRs linked separately.
- [ ] Rollback procedure executable: exact command, owner, duration — in the runbook.
- [ ] Triggers objective and written before deploy — quoted in the deploy doc.
- [ ] Rollback and kill-switch both exercised in staging — run/log linked.

## Example

Deploy: replace `orders.status` string with a state-machine enum + new `state` column. Plan produced: Deploy 1 adds nullable `state`, dual-write behind flag `orders_state_v2` (off). Deploy 2: flag to 5% canary with gate "state/status mismatch metric == 0", backfill job (batched 10k rows, resumable, ran 6h). Deploy 3, next week: reads switch to `state`, `status` still written. Deploy 4, three weeks later: contract — stop writing `status`; column dropped a month after. During Deploy 2 the mismatch metric fired (legacy cron wrote `status` directly, bypassing the ORM) — flag off in 20 seconds, cron fixed, re-canaried next day. No rollback ever touched the schema.

## Related skills

- `incident-triage` — the consumer of step 5's triggers and procedures.
- `environment-parity` — staging that can actually rehearse step 7.
- `error-handling-strategy` — the mismatch-metric pattern is its "loud fallback" rule.
