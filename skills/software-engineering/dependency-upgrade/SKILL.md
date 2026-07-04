---
name: dependency-upgrade
description: >
  Use when upgrading a library, framework, or runtime version — planning
  order, reading changelogs for breakage, staging the rollout, keeping a
  rollback path. Triggers: "upgrade to v3", "bump this dependency",
  "dependabot PR", "major version upgrade", "update node/python/java
  version", "is it safe to upgrade". NOT for emergency CVE patching
  triage (see dependency-vulnerability-audit first — then return here
  for mechanics).
---

# Dependency Upgrade

## Overview

Upgrades break things through changed defaults, removed APIs, and transitive surprises — mostly in ways the test suite wasn't written to see. Reading the changelog before touching anything, and moving one dependency per revertible PR, converts upgrade risk into routine.

## When to Use

- Major-version upgrades of libraries, frameworks, or language runtimes.
- Triaging a pile of Dependabot/Renovate PRs.

**When NOT to use:**
- Active CVE response — `dependency-vulnerability-audit` decides urgency and scope first; this skill executes the bump.

## Prerequisites

- Lockfile under version control (`package-lock.json`, `poetry.lock`, `go.sum`...). No lockfile = fix that first; upgrades without one aren't reproducible or revertible.
- CI that runs the test suite on PRs.

## The Workflow

1. **Read the changelog/migration guide before touching anything.** Search it for: "BREAKING", "removed", "deprecated", "default changed". Grep your codebase for each removed/changed API *now* — this tells you if it's a 20-minute bump or a two-week migration, before you've committed to either. This applies to minors too when the dependency is load-bearing: semver is a promise, not a guarantee.

2. **Check the dependency graph for conflicts.** `npm ls <pkg>` / `pipdeptree -r -p <pkg>` / `mvn dependency:tree`. If another dependency pins the old major, upgrading yours means upgrading (or replacing) that one too — sequence those first.

3. **Upgrade one thing per PR.** A 40-package Dependabot batch that breaks CI is undiagnosable. Exception: lockstep families (e.g. all `@babel/*`, kernel+headers) move together by design. Warning: diff the lockfile itself — a regeneration can silently bump 200 transitive packages inside an "innocent" PR.

4. **Bridge major versions through deprecation warnings when offered.** The well-trodden path for frameworks: upgrade to the last minor of the old major, turn warnings into errors in tests (`python -W error::DeprecationWarning`, React StrictMode, Rails `config.active_support.deprecation = :raise`), fix everything, *then* bump the major. Most breakage gets caught with old behavior still running. Keep the deprecation fixes and the major bump in separate PRs — when something breaks, you must know which half did it.

5. **Test beyond the unit suite.** Unit tests mock exactly the thing you changed. Run integration tests, boot the app, exercise the feature that uses the dependency. For serialization/ORM/date libraries, diff actual output on fixed inputs across versions — behavior changes hide where APIs didn't change.

6. **Stage the rollout like a risky deploy** (because it is one): canary or one service first, watch error rates and latency for a day, then fleet. Runtime and framework upgrades especially — perf regressions don't show in tests; connection-pool, GC, and TLS behavior changes only show under real traffic.

7. **Keep the rollback trivially cheap, and leave a trail.** The revert of the lockfile commit must be deployable in minutes — don't stack feature work on the upgrade commit until it has soaked. PR description records: versions from→to, breaking changes handled, what was tested, soak result. The 2am roll-backer needs this.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "It's a patch release, no need to read anything" | Patch releases change defaults and tighten validation regularly. For load-bearing dependencies the changelog read is two minutes; the surprise is an incident. |
| "Batch all the Dependabot PRs — one CI run, done" | When the batch breaks CI, which of 40 packages did it? Undiagnosable by construction. One-per-PR is slower per merge and faster per month. |
| "Tests pass, ship the major bump" | The suite mocks the exact thing that changed, and perf/GC/TLS deltas don't run in CI at all. Step 5's beyond-unit testing and step 6's soak exist because green-suite majors have broken plenty of prods. |
| "We'll do deprecation fixes and the major in one PR — fewer reviews" | One PR means one undiagnosable failure surface. The two-PR split is precisely what makes the breakage attributable. |
| "We're too busy to upgrade this quarter" | Ten majors behind is a rewrite; one major behind is a chore. Deferral doesn't reduce the work — it compounds it and removes the option of doing it calmly. |
| "The framework team tested it; that's what release candidates are for" | They tested their code against their assumptions. Your weird usage of their edge case is exactly what RC testing doesn't cover. |

## Red Flags

- Upgrade PR with no mention of the changelog or breaking changes — nobody read it.
- Lockfile diff shows hundreds of transitive bumps in a "single-package" PR.
- Major bump merged the same day it was opened, straight to full fleet.
- Feature commits stacked on top of an unsoaked upgrade.
- The same dependency upgraded twice in a week (the first attempt broke; the second is a guess).
- Deprecation warnings suppressed instead of fixed.

## Verification

- [ ] Changelog read; breaking-change list with per-item disposition in the PR description.
- [ ] Codebase grepped for each removed/changed API — hits and fixes listed.
- [ ] Lockfile diff reviewed; transitive changes acknowledged in the PR.
- [ ] Beyond-unit evidence attached: integration run, app boot, or output-diff on fixed inputs as applicable.
- [ ] Soak completed per plan (canary/staging duration and metrics linked) before fleet.
- [ ] Rollback path stated and unblocked (no stacked commits) — one-line revert instruction in the PR.

## Example

Task: SQLAlchemy 1.4 → 2.0. Step 1 found the migration guide's removed-API list; grep showed 63 uses of legacy `Query`. Plan: stay on 1.4, enable `SQLALCHEMY_WARN_20=1`, turn warnings into CI errors, burn down the 63 call sites over four small PRs, then a one-line bump PR. Bump PR soaked on the staging replica for two days; caught one behavior change (autobegin) the suite missed because tests used explicit transactions. Total: five revertible PRs, no rollback needed.

## Related skills

- `dependency-vulnerability-audit` — when the driver is a CVE, triage exploitability first.
- `deployment-rollback-plan` — making step 7's rollback path real.
- `technology-evaluation` — when the upgrade cost is high enough to justify replacing the dependency instead.
