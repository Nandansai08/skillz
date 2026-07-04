---
name: environment-parity
description: >
  Use when dev/staging/prod behave differently and you need them aligned —
  12-factor config discipline, what must match vs may differ, closing
  "works in staging" gaps. Triggers: "works in staging but not prod",
  "works on my machine", "staging is nothing like prod", "environment
  config", "parity between environments". NOT for debugging the specific
  divergent incident right now (see production-debugging) — this skill
  fixes the class.
---

# Environment Parity

## Overview

"Works in staging, fails in prod" means staging tests a different program — different artifact, different config keys, different data shape. One artifact promoted everywhere, divergences classified must-match/may-differ/must-differ, and mechanical verification hold the environments honest.

## When to Use

- A bug reproduces in one environment and not another.
- Designing the config/environment story for a new service.

**When NOT to use:**
- The divergent incident burning right now — `production-debugging`; return here for the class fix.

## Prerequisites

- List of environments in play and how each is built/deployed today.

## The Workflow

1. **Same artifact everywhere — the foundation rule.** One image/binary built once in CI, promoted dev → staging → prod. Environments differ *only* by injected configuration. If prod is built from a different branch, flag set, or build command than staging, staging tests a different program; fix this before anything else. Corollary: no `if env == "production"` branches in application code — feature flags and injected config express differences; env-name conditionals hide them inside the artifact.

2. **Split config into the three buckets:**
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

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "SQLite in dev is more convenient than running Postgres" | Dialect drift is a permanent bug tax paid on every feature. Containers made the convenience argument obsolete a decade ago — `docker compose up` is the new convenient. |
| "Staging is for testing, so a few manual tweaks are fine" | Staging-as-junk-drawer (hand fixes, debug flags, hand-made data) means it validates nothing. If rebuilding staging from code is scary, that fear is the parity debt itself. |
| "The env-name conditional is just one small branch" | It forks the artifact's behavior invisibly — the exact thing single-artifact promotion exists to prevent. One conditional becomes twelve, and staging tests a program prod never runs. |
| "Different Python patch versions can't matter" | The example's incident was exactly a runtime-minor delta. 'Can't matter' is a hypothesis; the must-match list plus the mechanical diff replaces it with a fact. |
| "We can't afford prod-scale staging data" | You don't need prod SCALE for most bugs — you need prod SHAPE (distributions, weird values, volume on hot tables). Seeded shape costs a script; the class of bugs it catches costs incidents. |
| "The parity audit is done — we fixed everything last year" | Divergence is entropy: every hotfix, console tweak, and version bump re-opens gaps. Only the scheduled mechanical check (step 6) holds; one-time audits decay silently. |

## Red Flags

- Different build commands/branches per environment.
- `if env ==` conditionals in application code.
- Env-var keys present in prod with no staging counterpart.
- Staging DB with 200 rows validating queries that run against 40M.
- "Works on my machine" arising from undocumented local toolchain versions.
- Parity spreadsheet dated last year; no CI job enforcing any of it.

## Verification

- [ ] Single-artifact promotion confirmed: same digest/hash deployed across envs — deploy logs linked.
- [ ] Zero env-name conditionals in app code — grep output attached.
- [ ] Divergence inventory exists with must/may/must-differ classification — every must-match violation ticketed.
- [ ] Env-var key-diff running in CI — job linked; current diff empty or dispositioned.
- [ ] Data-shape story implemented (refresh schedule or seed scripts) — linked.
- [ ] Residual gaps named in writing, with their canary/observability compensation stated.

## Example

Recurring "staging green, prod 500s" incidents. Inventory (step 3) found: prod on Python 3.11 vs staging 3.12, staging built from `develop` branch (violating step 1), prod had `PAYMENT_RETRIES=5` with no staging equivalent (key-diff would have caught it — that untested retry path was the 500). Fixes: single-artifact promotion pipeline, env-var key diff in nightly CI, version pins unified in one file, staging DB re-seeded weekly with anonymized prod-shaped data. The next payment-path bug reproduced in staging first — first time ever.

## Related skills

- `production-debugging` — when the divergence is burning right now.
- `deployment-rollback-plan` — canaries as the compensation for residual gaps.
- `secrets-management` — the must-differ bucket's handling.
