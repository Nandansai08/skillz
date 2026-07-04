---
name: dockerfile-optimization
description: >
  Use when writing or fixing a Dockerfile — layer caching, multi-stage
  builds, image size, build speed, container security basics. Triggers:
  "optimize this Dockerfile", "image is huge", "docker build is slow",
  "multi-stage build", "reduce image size", "Dockerfile best practices".
  NOT for orchestration concerns — probes, limits, rollout strategy are
  deploy config (see deployment-rollback-plan).
---

# Dockerfile Optimization

## Overview

Most slow builds and bloated images come from two mistakes: layers ordered against their change frequency, and build-time tooling shipped in the runtime image. Layer ordering, multi-stage builds, and a security floor fix the class.

## When to Use

- Writing a Dockerfile for a service, or fixing one that's slow/huge.
- Reviewing container changes in a PR.

**When NOT to use:**
- Probes, resource limits, rollout strategy — deploy configuration, `deployment-rollback-plan` territory.

## Prerequisites

- BuildKit enabled (default in modern Docker; `DOCKER_BUILDKIT=1` otherwise).

## The Workflow

1. **Order layers by change frequency — the single highest-leverage rule.** Least→most volatile: base image → system packages → dependency manifests + install → application code. Copy manifests *alone* before code:
   ```dockerfile
   COPY package.json package-lock.json ./
   RUN npm ci
   COPY . .
   ```
   Code edits then reuse the cached dependency layer instead of reinstalling on every build.

2. **Multi-stage: build heavy, ship light.** Compilers, dev headers, and node_modules-with-devdeps stay in the builder; the final stage copies artifacts only:
   ```dockerfile
   FROM node:22 AS build
   WORKDIR /app
   COPY package*.json ./
   RUN npm ci
   COPY . .
   RUN npm run build && npm prune --omit=dev

   FROM node:22-slim
   WORKDIR /app
   COPY --from=build /app/dist ./dist
   COPY --from=build /app/node_modules ./node_modules
   USER node
   CMD ["node", "dist/server.js"]
   ```

3. **Pick the smallest base that doesn't fight you.** `-slim` variants are the sane default; `alpine` only if you've verified musl compatibility (Python wheels and glibc-linked binaries break); `distroless`/`scratch` for static Go/Rust binaries. Pin by digest or at least minor tag (`node:22.4-slim`, never bare `latest`). No `apt-get upgrade` in the Dockerfile — unpinned drift makes images differ by build day; take updates by bumping the base tag deliberately.

4. **Clean up inside the same RUN.** A later `rm` doesn't shrink an earlier layer:
   ```dockerfile
   RUN apt-get update && apt-get install -y --no-install-recommends curl \
       && rm -rf /var/lib/apt/lists/*
   ```
   Combine only where cleanup requires it — over-chaining unrelated commands kills cache granularity (layer *count* is nearly free; invalidation granularity is what costs).

5. **`.dockerignore` before anything else.** `.git`, `node_modules`, build output, `.env`, test data. Without it, `COPY . .` invalidates cache on any file change and ships secrets you forgot about. Check: `docker build` context size printed at build start — hundreds of MB means the ignore file is wrong.

6. **Use BuildKit cache mounts for package managers** — persistent cache without layer bloat:
   ```dockerfile
   RUN --mount=type=cache,target=/root/.npm npm ci
   ```
   In CI, add registry-backed layer caching (`--cache-from type=registry,...` with buildx) so runners share cache.

7. **Security floor, non-negotiable:** run as non-root (`USER`), no secrets in ENV/ARG/layers (BuildKit `--mount=type=secret` for build-time needs — ARGs persist in `docker history`), and a vulnerability scan in CI (`trivy image`, `grype`) gating on critical CVEs.

8. **Verify with `docker history <image>` and `dive`.** Layer sizes tell you where the bytes went; dive shows wasted files. Measure before and after — optimizations you didn't measure are rearrangements.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "COPY . . first is simpler — one line" | And it reinstalls every dependency on every code edit, forever. The manifests-first split is two lines and the #1 build-speed fix in the genre. |
| "The API key goes in a build ARG, it's temporary" | ARGs persist in image history — `docker history` prints them. A leaked registry image is a leaked secret; `--mount=type=secret` exists for exactly this. |
| "Alpine is smallest, so alpine" | Until the musl/glibc segfault in a native module eats a week. Slim is ~30MB more and boring; smallest-that-doesn't-fight-you is the actual rule. |
| "We'll add the scanner once the image stabilizes" | Base images accrue CVEs while 'stabilizing.' The example's trivy gate caught an OpenSSL critical in week one — in the base everyone assumed was fine. |
| "Root in the container is fine, it's isolated" | Container escape vulns exist, and non-root is one line. The floor is called a floor because there's no cheaper mitigation anywhere in security. |
| "One giant RUN chain minimizes layers — best practice" | That advice died with layer-count costs. Chaining unrelated commands means any change re-runs all of them; granularity is the asset now. |

## Red Flags

- Build context measured in hundreds of MB (no/broken `.dockerignore`).
- `COPY . .` above the dependency install.
- `latest` or bare-major base tags in production Dockerfiles.
- Secrets visible in `docker history` output.
- Single-stage builds shipping compilers/devDependencies to production.
- No scan gate; base image unbumped for months.

## Verification

- [ ] Rebuild-after-code-edit reuses the dependency layer — build log showing cache hits attached.
- [ ] Image size before/after measured — numbers in the PR.
- [ ] Build context size sane (KB–low MB) — build output line quoted.
- [ ] `docker history` clean of secrets; runtime user non-root — outputs attached.
- [ ] Scanner gate green (or criticals dispositioned) — CI link.
- [ ] Base pinned to at least minor tag — Dockerfile diff shows it.

## Example

Python API image: 1.9GB, 11-min builds. Findings: `python:3.12` full base, `COPY . .` first line, no ignore file (700MB context incl. `.git` and test fixtures), pip cache baked into a layer. After: `.dockerignore` (context → 4MB), manifests-first ordering with `--mount=type=cache` pip install, multi-stage dropping the compiler toolchain, `python:3.12-slim` final, `USER app`. Result: 210MB, 50-second rebuild on code change, trivy gate added — caught an OpenSSL critical in the old pinned base the first week.

## Related skills

- `ci-pipeline-design` — where the registry cache and scan gates live.
- `secrets-management` — the runtime half of the secrets rule.
- `dependency-upgrade` — bumping the pinned base image deliberately.
