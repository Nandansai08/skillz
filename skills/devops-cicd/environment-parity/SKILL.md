---
name: environment-parity
description: >
  Use when dev/staging/prod behave differently and you need them aligned —
  12-factor config discipline, what must match vs may differ, closing
  "works in staging" gaps. Triggers: "works in staging but not prod",
  "works on my machine", "staging is nothing like prod", "environment
  config", "parity between environments".
---

# Environment Parity

## When to use this skill
- A bug reproduces in one environment and not another.
- Designing the config/environment story for a new service.
- NOT for debugging the specific divergent incident right now — that's `production-debugging`; this skill fixes the class.

## Prerequisites
- List of environments in play and how each is built/deployed today.

## Workflow

1. **Same artifact everywhere — the foundation rule.** One image/binary built once in CI, promoted dev → staging → prod. Environments differ *only* by injected configuration. If prod is built from a different branch, flag set, or build command than staging, staging tests a different program; fix this before anything else.

2. **Split config into the three 12-factor-ish buckets:**
   - **Code-shipped** (same everywhere): timeouts, feature defaults, tuning constants. Lives in the repo.
   - **Environment config** (differs by design): URLs, DB hosts, pool sizes, log levels. Injected env vars / config service; every value has a *reason* to differ.
   - **Secrets:** injected per `secrets-management`, never in either of the above.
   Then audit: any env-var that *could* be code-shipped should be — every unnecessary knob is a parity risk.

3. **Inventory the divergences and classify each: must-match / may-differ / must-differ.**
   Must-match: runtime version, dependency versions (same lockfile — guaranteed by step 1), DB engine major.minor, timezone (UTC everywhere), middleware/proxy chain shape.
   May-differ: scale (replica counts), data volume.
   Must-differ: credentials, third-party endpoints (sandbox vs live), alert routing.
   Every "must-match" that currently doesn't is a ticket with an owner.

4. **Close the data-shape gap deliberately.** Staging with 100 rows can't reproduce prod query plans, timeouts, or pagination bugs. Options in order of preference: anonymized/synthesized prod-shaped dataset refreshed on schedule; production data *sampling* with PII scrubbing (legal review required); at minimum, seed scripts that generate prod-*scale* volumes for the hot tables.

5. **Make dev parity cheap or it won't happen.** Containers for the service dependencies (same Postgres/Redis tags as prod — pin once, reference everywhere: one `versions` file that Dockerfiles, compose, and CI all read), devcontainer or nix/mise for toolchain versions. Docs-based parity ("install Postgres 16") drifts within a month.

6. **Verify parity mechanically, on schedule.** A CI job that diffs: runtime versions across envs (hit each `/health` endpoint's version info), env-var *keys* (not values) across environments — a key present in prod but absent in staging is an untested code path; infra via IaC (envs built from the same modules with different tfvars, so drift shows in `terraform plan`).

7. **Accept the gaps you can't close, and compensate.** Real traffic patterns, third-party live behavior, and full prod scale never fully replicate. Name these residual gaps and cover them with canary deploys + production observability rather than pretending staging catches everything (see `deployment-rollback-plan` step 6).

## Common pitfalls
- `if env == "production"` branches in application code — behavior forks the artifact rule can't save you from. Feature flags and injected config express the difference; env-name conditionals hide it.
- Staging as the team's junk drawer: manual hotfixes, debug flags left on, hand-created test data. Rebuild staging from code on schedule; if rebuilding it is scary, that's the parity debt talking.
- Matching software but not shape: prod runs 40 replicas behind a load balancer, staging runs 1 — connection-pool exhaustion, sticky-session, and thundering-herd bugs are invisible until prod.
- SQLite/H2 in dev "for convenience" vs Postgres in prod. Dialect drift is a permanent bug tax; containers made this excuse obsolete (`integration-test-strategy` step 3).
- Parity audit done once, never re-run. Divergence is entropy; only the scheduled check (step 6) holds it.

## Example
Recurring "staging green, prod 500s" incidents. Inventory (step 3) found: prod on Python 3.11 vs staging 3.12, staging built from `develop` branch (violating step 1), prod had `PAYMENT_RETRIES=5` with no staging equivalent (key-diff would have caught it — that untested retry path was the 500). Fixes: single-artifact promotion pipeline, env-var key diff in nightly CI, version pins unified in one file, staging DB re-seeded weekly with anonymized prod-shaped data. The next payment-path bug reproduced in staging first — first time ever.

## Related skills
- `production-debugging` — when the divergence is burning right now.
- `deployment-rollback-plan` — canaries as the compensation for residual gaps.
- `secrets-management` — the must-differ bucket's handling.
