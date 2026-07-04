---
name: ci-pipeline-design
description: >
  Use when designing or speeding up a CI pipeline — stage ordering,
  fail-fast, caching, parallelism, what runs on PR vs merge vs nightly.
  Triggers: "CI is too slow", "design the pipeline", "speed up the build",
  "what should run on every PR", "optimize CI". NOT for authoring the
  workflow syntax itself (see github-actions-authoring) and NOT for
  shrinking the e2e suite the pipeline runs (see e2e-test-triage).
---

# CI Pipeline Design

## Overview

CI feedback past ten minutes changes developer behavior for the worse: batched commits, context switching, admin-merges. Pipeline design is the discipline of buying that ten minutes with ordering, caching, sharding, and trigger routing — using measurements, not folklore.

## When to Use

- Building CI for a new repo or overhauling a slow one.
- Deciding which checks gate merges vs run async.

**When NOT to use:**
- Actions/workflow syntax mechanics — `github-actions-authoring`.
- The e2e suite's own size and stability — `e2e-test-triage` (this skill consumes its smoke set).

## Prerequisites

- Timing data for current jobs (any CI shows per-step duration). Optimizing without measurements rearranges guesses.

## The Workflow

1. **Set the target: PR feedback ≤ 10 minutes.** Beyond that, developers context-switch, batch commits, and stop trusting CI. Every decision below serves this number.

2. **Order stages cheapest-first, fail-fast.** Lint/format/typecheck (seconds) → unit tests (a minute) → build → integration tests → e2e smoke. A typo shouldn't wait 8 minutes of integration tests to be reported. Within a stage, run independent jobs in parallel; across stages, only chain what truly depends.

3. **Cache the three expensive things, keyed correctly:**
   - **Dependencies:** key on lockfile hash (`hashFiles('**/package-lock.json')`) — not on branch, not static.
   - **Build artifacts/compilers:** language-native caches (Gradle build cache, `sccache`, Turborepo/Nx remote cache, Docker layer cache via registry or buildx).
   - **Test infrastructure:** pre-pulled container images.
   Verify cache *hit rate* after setup — a mis-keyed cache silently downloads everything while reporting "cache restored."

4. **Parallelize tests by measured shards.** Split by historical timing (not file count) so shards finish together: pytest-split, Jest `--shard`, Playwright sharding, CI matrix. Four even 3-min shards beat one 12-min job; twenty 40-s shards mostly pay startup overhead — find the knee.

5. **Route work by trigger, not habit:**
   - **PR:** lint, types, units, integration, e2e smoke set — must fit the 10-min budget.
   - **Merge to main:** full e2e, artifact publish, deploy to staging.
   - **Nightly:** mutation tests, long property runs, dependency audit, performance benchmarks.
   - **Path-filtered:** monorepos skip unaffected packages (`turbo --filter=[HEAD^]`, Bazel/Nx affected-graph). Biggest single win in multi-service repos.
   Warning: deploy stages and secrets must trigger only from protected branches — pipeline design is also a security boundary for fork PRs.

6. **Make main's health enforce itself.** Required checks on the protected branch; merge queue (GitHub merge queue, Bors-style) when concurrent merges break each other's green; auto-revert or page-owner policy for a red main. A red main that stays red for a day teaches everyone to ignore CI.

7. **Track two pipeline metrics monthly:** p50/p95 PR-feedback time, and false-failure rate (runs failed for non-code reasons: flakes, infra, cache). Feedback time creeping up and false failures above ~1% are the maintenance signals; budget the fix time before trust erodes.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "One serial job is simpler to maintain" | The 25-minute monolith has no fail-fast, no per-step timing, and no parallelism — it's simpler the way a traffic jam is simpler than traffic lights. Stage structure IS the maintainability. |
| "Run everything on every PR — safety first" | Safety that costs 40 minutes gets bypassed with admin merges, which is zero safety. The routed design (step 5) runs everything — just not all of it per-PR. |
| "The cache is configured, so it's working" | Mis-keyed caches restore stale-nothing while logging success. Only the hit-rate metric knows; check it after every cache change. |
| "Devs can just rerun on flaky failures" | Ten reruns a day is an unbudgeted engineer, and each rerun teaches everyone that red might mean nothing. The false-failure metric (step 7) makes this cost visible. |
| "We'll speed it up when it gets really bad" | 'Really bad' arrives via 20 increments nobody owned. The monthly p95 review catches the creep while each cause is still one fix. |
| "Branch-keyed caching works fine" | It serves stale dependencies to PRs that pass, then break on main after merge — the worst possible failure location. Lockfile-hash keys exist for this. |

## Red Flags

- PR feedback p95 above 15 minutes and drifting up.
- Admin-merge/bypass usage becoming routine.
- Cache "restored" logs with install times unchanged.
- One mega-job whose internal step timings nobody can name.
- Red main persisting overnight without a page or revert.
- Fork PRs able to reach jobs holding secrets.

## Verification

- [ ] PR p50/p95 feedback time measured before and after — numbers in the change description.
- [ ] Cache hit rates verified post-change (log excerpts or metrics linked).
- [ ] Shards balanced by timing — slowest vs fastest shard delta shown.
- [ ] Trigger routing table documented (PR/merge/nightly/path-filtered) — config linked.
- [ ] Fork-PR secret exposure checked: deploy/secret jobs unreachable from forks — config shown.
- [ ] False-failure rate baseline established and dashboarded.

## Example

Node monorepo, 28-min serial pipeline, one job. Measured: install 6m, lint 1m, units 9m, e2e 12m. Redesign: lockfile-keyed dependency cache (install → 40s on hit), lint+typecheck parallel with unit shards (4 shards by timing, 2.5m), e2e cut to 8-test smoke on PR with the full set post-merge, Turborepo path filtering so docs-only PRs run in 90s. PR p95: 7m20s. False-failure rate surfaced at 4% — traced to a shared staging DB in integration tests, fixed with per-shard schemas (see `integration-test-strategy`). Merge queue added after two same-day semantic conflicts on main.

## Related skills

- `github-actions-authoring` — implementing this design in Actions syntax.
- `e2e-test-triage` — producing the smoke set step 5 needs.
- `dockerfile-optimization` — the image builds this pipeline caches.
