---
name: release-versioning
description: >
  Use when setting up or fixing how a project versions and releases —
  semver decisions, tagging, changelogs, release automation. Triggers:
  "what version should this be", "is this a breaking change", "set up
  releases", "automate versioning", "tag a release", "conventional
  commits". NOT for writing user-facing release notes prose (see
  release-notes) — this is the mechanics.
---

# Release Versioning

## Overview

Version numbers are promises to consumers: major means "you must act," minor means "you may," patch means "you needn't." The machinery — automated bumps, immutable tags, generated-then-edited changelogs — exists to keep those promises cheap to make and hard to break.

## When to Use

- Establishing the release process for a library, service, or CLI.
- Deciding whether a specific change bumps major/minor/patch.

**When NOT to use:**
- The narrative announcement layer — `release-notes`.

## Prerequisites

- Clarity on who consumes the releases: external developers (semver is a contract), internal services (looser), or end-users (marketing versions may differ from build versions).

## The Workflow

1. **Decide what "the public API" is, in writing.** Semver is meaningless until you define what's covered: exported functions? CLI flags and output format? Config file schema? HTTP endpoints? Wire formats? A `VERSIONING.md` sentence like "the public API is everything under `pkg/`, the CLI flags, and the config schema — CLI *output* format is not covered" prevents most breaking-change arguments before they start.

2. **Bump by consumer impact, not effort.** Major: an existing consumer must change something to upgrade (removed/renamed API, changed default behavior, raised minimum runtime). Minor: new capability, existing consumers unaffected. Patch: bug fixes only — and if consumers depended on the buggy behavior, a fix can still be breaking (announce it). Rewriting the internals for a year = patch, if the contract held; renaming one public function = major.

3. **For services (no external installers), simplify:** continuous deployment with build metadata (`2026.07.03-a1b2c3d` or plain SHA) usually beats semver theater — services have one deployed version, not a compatibility matrix. Semver stays for the service's *client libraries* and published API versions (`api-design-rest` step 6).

4. **Automate the bump from commit messages.** Conventional Commits (`feat:`, `fix:`, `feat!:`/`BREAKING CHANGE:`) + release tooling (`release-please`, `semantic-release`, `changesets` for monorepos — changesets preferred where PRs should declare their own impact). The human decision stays "is this breaking?" — encoded once in the commit/changeset; version arithmetic, tag, and changelog generate from it. Single source of truth for the version number, propagated by tooling — hand-bumping three files leaves one lying.

5. **Tag protocol:** annotated tags (`git tag -a v2.3.0 -m "..."`), `v`-prefixed, created by CI on the release commit — never hand-pushed from laptops. The tag triggers the publish workflow (`github-actions-authoring`). Tags are immutable: a broken release gets a new patch version, never a moved tag.

6. **Changelog: keep-a-changelog format, generated then edited.** Sections Added/Changed/Fixed/Removed/Deprecated/Security, newest first, each entry linking the PR. Auto-generation gives completeness; a 2-minute human edit gives sense (`changelog-writing` owns the editing craft).

7. **Pre-releases, deprecation, and the 0.x question:** `-rc.1`/`-beta.1` suffixes for testing releases; deprecate in a minor (warn, document the replacement), remove in the next major, with the support window for old majors stated. And 0.x: either commit to 1.0 early and honor it, or state loudly that 0.x minors may break — perpetual 0.x on a widely-used library is a versioning bug.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "It's a tiny rename — minor bump at most" | Size is irrelevant; one renamed parameter breaks every caller. Consumer impact is the only axis, and a rename scores major every time. |
| "Nobody depends on the buggy behavior we just fixed" | Hyrum's Law says someone does. Behavioral fixes to long-standing bugs get announced like the breaking changes they may be. |
| "Marketing wants 5.0 — just skip the versions" | The jump destroys the semantic signal every consumer's tooling relies on. Give marketing a product version; keep the artifact version honest. |
| "Lockstep releases keep the monorepo simple" | Twelve unchanged packages bumping majors teach consumers that your majors mean nothing — the signal decays to noise. Independent versioning unless genuinely coupled. |
| "I'll hand-push the tag, CI is slow today" | Laptop tags skip checks, signing, and reproducibility — and moved-tag temptation follows. The tag protocol has no fast lane. |
| "We're 0.x, breaking changes are allowed, no announcement needed" | Allowed by spec, unread by consumers. Two years at 0.9 with breaking minors is a 1.0 refusing to happen — commit or warn loudly. |

## Red Flags

- The version number differing between `package.json`, the CLI's `--version`, and the tag.
- Changelog entries reading "wip", "fix", "address review" verbatim from git log.
- A moved or deleted tag anywhere in history.
- Breaking changes discovered by consumers, not announced by the changelog.
- "What's our public API?" answered differently by two maintainers.
- Marketing-driven version jumps in the artifact version stream.

## Verification

- [ ] `VERSIONING.md` (or equivalent) defines the covered API surface — link.
- [ ] Bump automation wired: version, tag, and changelog generate from commit/changeset metadata — release PR linked.
- [ ] One version source of truth; other occurrences generated — demonstrated by the release diff.
- [ ] Tags annotated, CI-created, immutable — protocol visible in the workflow.
- [ ] Latest changelog human-edited (not raw commit dump) — diff between generated and shipped shown.
- [ ] Deprecation policy and old-major support window stated — link.

## Example

CLI tool at 0.9.x for two years, breaking users each minor. Applied: `VERSIONING.md` defining the contract (flags + exit codes + machine-readable `--json` output covered; human-readable output not), cut 1.0.0, changesets + release-please wired so PRs declare impact, CI-created signed tags triggering publish. Three months later a PR renamed `--out` to `--output`: changeset forced the "breaking?" question at review time → shipped as 2.0.0 with `--out` kept as deprecated alias for one major. Zero surprise-breakage issues since; changelog edits take ~3 minutes per release.

## Related skills

- `release-notes` — the human-facing narrative on top of step 6's changelog.
- `changelog-writing` — the editing craft for the generated changelog.
- `github-actions-authoring` — the tag-triggered publish workflow.
- `api-design-rest` — versioning the HTTP contract specifically.
- `dependency-upgrade` — this skill seen from the consumer's side.
