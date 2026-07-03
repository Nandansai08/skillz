---
name: dependency-upgrade
description: >
  Use when upgrading a library, framework, or runtime version — planning the
  order, reading changelogs for breakage, staging the rollout, keeping a
  rollback path. Triggers: "upgrade to v3", "bump this dependency",
  "dependabot PR", "major version upgrade", "update node/python/java version",
  "is it safe to upgrade".
---

# Dependency Upgrade

## When to use this skill
- Major-version upgrades of libraries, frameworks, or language runtimes.
- Triaging a pile of Dependabot/Renovate PRs.
- NOT for emergency CVE patches — see `dependency-vulnerability-audit` for the triage-first flow; come back here for the mechanics.

## Prerequisites
- Lockfile under version control (`package-lock.json`, `poetry.lock`, `go.sum`...). No lockfile = fix that first; upgrades without one aren't reproducible or revertible.
- CI that runs the test suite on PRs.

## Workflow

1. **Read the changelog/migration guide before touching anything.** Search it for: "BREAKING", "removed", "deprecated", "default changed". Grep your codebase for each removed/changed API *now* — this tells you if it's a 20-minute bump or a two-week migration, before you've committed to either.

2. **Check the dependency graph for conflicts.** `npm ls <pkg>` / `pipdeptree -r -p <pkg>` / `mvn dependency:tree`. If another dependency pins the old major, upgrading yours means upgrading (or replacing) that one too — sequence those first.

3. **Upgrade one thing per PR.** A 40-package Dependabot batch that breaks CI is undiagnosable. Exception: lockstep families (e.g. all `@babel/*`, kernel+headers) move together by design.

4. **Bridge major versions through deprecation warnings when offered.** The well-trodden path for frameworks: upgrade to the last minor of the old major, turn warnings into errors in tests (`python -W error::DeprecationWarning`, React StrictMode, Rails `config.active_support.deprecation = :raise`), fix everything, *then* bump the major. Most breakage gets caught with old behavior still running.

5. **Test beyond the unit suite.** Unit tests mock exactly the thing you changed. Run integration tests, boot the app, exercise the feature that uses the dependency. For serialization/ORM/date libraries, diff actual output on fixed inputs across versions — behavior changes hide where APIs didn't change.

6. **Stage the rollout like a risky deploy** (because it is one): canary or one service first, watch error rates and latency for a day, then fleet. Runtime and framework upgrades especially — perf regressions don't show in tests.

7. **Keep the rollback trivially cheap.** The revert of the lockfile commit must be deployable in minutes. Don't stack feature work on top of the upgrade commit until it has soaked.

8. **Leave a trail.** PR description records: versions from→to, breaking changes handled, what was tested, soak result. The next upgrader (or the person rolling back at 2am) needs this.

## Common pitfalls
- Trusting semver. Minor and patch releases break things regularly (changed defaults, tightened validation). The changelog read in step 1 applies to minors too when the dependency is load-bearing.
- Upgrading transitive deps invisibly. A lockfile regeneration can bump 200 transitive packages inside an "innocent" PR — diff the lockfile and look.
- Fixing deprecations and bumping the major in one PR. When it breaks, you can't tell which half did it.
- Letting upgrades pile up for a year. Ten majors behind is a rewrite, one major behind is a chore. Schedule the chore.
- Skipping the soak because "tests pass." Connection-pool, GC, and TLS behavior changes only show under real traffic.

## Example
Task: SQLAlchemy 1.4 → 2.0. Step 1 found the migration guide's removed-API list; grep showed 63 uses of legacy `Query`. Plan: stay on 1.4, enable `SQLALCHEMY_WARN_20=1`, turn warnings into CI errors, burn down the 63 call sites over four small PRs, then a one-line bump PR. Bump PR soaked on the staging replica for two days; caught one behavior change (autobegin) the suite missed because tests used explicit transactions. Total: five revertible PRs, no rollback needed.

## Related skills
- `dependency-vulnerability-audit` — when the driver is a CVE, triage exploitability first.
- `deployment-rollback-plan` — making step 7's rollback path real.
- `technology-evaluation` — when the upgrade cost is high enough to justify replacing the dependency instead.
