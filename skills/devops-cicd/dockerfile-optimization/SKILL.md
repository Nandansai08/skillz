---
name: dockerfile-optimization
description: >
  Use when writing or fixing a Dockerfile — layer caching, multi-stage
  builds, image size, build speed, container security basics. Triggers:
  "optimize this Dockerfile", "image is huge", "docker build is slow",
  "multi-stage build", "reduce image size", "Dockerfile best practices".
---

# Dockerfile Optimization

## When to use this skill
- Writing a Dockerfile for a service, or fixing one that's slow/huge.
- Reviewing container changes in a PR.
- NOT for orchestration concerns (probes, limits, rollout) — that's deploy config, see `deployment-rollback-plan`.

## Prerequisites
- BuildKit enabled (default in modern Docker; `DOCKER_BUILDKIT=1` otherwise).

## Workflow

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

3. **Pick the smallest base that doesn't fight you.** `-slim` variants are the sane default; `alpine` only if you've verified musl compatibility (Python wheels and glibc-linked binaries break); `distroless`/`scratch` for static Go/Rust binaries. Pin by digest or at least minor tag (`node:22.4-slim`, never bare `latest`).

4. **Clean up inside the same RUN.** A later `rm` doesn't shrink an earlier layer:
   ```dockerfile
   RUN apt-get update && apt-get install -y --no-install-recommends curl \
       && rm -rf /var/lib/apt/lists/*
   ```
   Combine only where cleanup requires it — over-chaining unrelated commands kills cache granularity.

5. **`.dockerignore` before anything else.** `.git`, `node_modules`, build output, `.env`, test data. Without it, `COPY . .` invalidates cache on any file change and ships secrets you forgot about. Check: `docker build` context size printed at build start — hundreds of MB means the ignore file is wrong.

6. **Use BuildKit cache mounts for package managers** — persistent cache without layer bloat:
   ```dockerfile
   RUN --mount=type=cache,target=/root/.npm npm ci
   RUN --mount=type=cache,target=/var/cache/apt apt-get install -y ...
   ```
   In CI, add registry-backed layer caching (`--cache-from type=registry,...` with buildx) so runners share cache.

7. **Security floor, non-negotiable:** run as non-root (`USER`), no secrets in ENV/ARG/layers (BuildKit `--mount=type=secret` for build-time needs), and a vulnerability scan in CI (`trivy image`, `grype`) gating on critical CVEs.

8. **Verify with `docker history <image>` and `dive`.** Layer sizes tell you where the bytes went; dive shows wasted files. Measure before and after — optimizations you didn't measure are rearrangements.

## Common pitfalls
- `COPY . .` before dependency install — every code change reinstalls all dependencies. The #1 slow-build cause.
- Secrets via build ARG "temporarily." ARGs persist in image history (`docker history` shows them); leaked registry image = leaked secret.
- `apt-get upgrade` in the Dockerfile — unpinned drift; images differ by build day. Take updates by bumping the base tag deliberately.
- Alpine adopted for size, then a week lost to a musl/glibc segfault in a native module. Slim is 30MB more and boring.
- One giant `RUN` chaining everything "for fewer layers." Layer *count* is nearly free; cache invalidation granularity is what costs.

## Example
Python API image: 1.9GB, 11-min builds. Findings: `python:3.12` full base, `COPY . .` first line, no ignore file (700MB context incl. `.git` and test fixtures), pip cache baked into a layer. After: `.dockerignore` (context → 4MB), manifests-first ordering with `--mount=type=cache` pip install, multi-stage dropping the compiler toolchain, `python:3.12-slim` final, `USER app`. Result: 210MB, 50-second rebuild on code change, trivy gate added — caught an OpenSSL critical in the old pinned base the first week.

## Related skills
- `ci-pipeline-design` — where the registry cache and scan gates live.
- `secrets-management` — the runtime half of the secrets rule.
- `dependency-upgrade` — bumping the pinned base image deliberately.
