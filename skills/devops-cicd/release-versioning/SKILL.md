---
name: release-versioning
description: >
  Use when setting up or fixing how a project versions and releases — semver
  decisions, tagging, changelogs, release automation. Triggers: "what version
  should this be", "is this a breaking change", "set up releases",
  "automate versioning", "tag a release", "conventional commits".
---

# Release Versioning

## When to use this skill
- Establishing the release process for a library, service, or CLI.
- Deciding whether a specific change bumps major/minor/patch.
- NOT for writing user-facing release notes prose — see `release-notes`; this is the mechanics.

## Prerequisites
- Clarity on who consumes the releases: external developers (semver is a contract), internal services (looser), or end-users (marketing versions may differ from build versions).

## Workflow

1. **Decide what "the public API" is, in writing.** Semver is meaningless until you define what's covered: exported functions? CLI flags and output format? Config file schema? HTTP endpoints? Wire formats? A `VERSIONING.md` sentence like "the public API is everything under `pkg/`, the CLI flags, and the config schema — CLI *output* format is not covered" prevents most breaking-change arguments before they start.

2. **Bump by consumer impact, not effort.** Major: an existing consumer must change something to upgrade (removed/renamed API, changed default behavior, raised minimum runtime). Minor: new capability, existing consumers unaffected. Patch: bug fixes only — and if consumers depended on the buggy behavior, a fix can still be breaking (announce it). Rewriting the internals for a year = patch, if the contract held; renaming one public function = major.

3. **For services (no external installers), simplify:** continuous deployment with build metadata (`2026.07.03-a1b2c3d` or plain SHA) usually beats semver theater — services have one deployed version, not a compatibility matrix. Semver stays for the service's *client libraries* and published API versions (`api-design-rest` step 6).

4. **Automate the bump from commit messages.** Conventional Commits (`feat:`, `fix:`, `feat!:`/`BREAKING CHANGE:`) + release tooling (`release-please`, `semantic-release`, `changesets` for monorepos — changesets preferred where PRs should declare their own impact). The human decision stays "is this breaking?" — encoded once in the commit/changeset, and the version arithmetic, tag, and changelog generate from it.

5. **Tag protocol:** annotated tags (`git tag -a v2.3.0 -m "..."`), `v`-prefixed, created by CI on the release commit — never hand-pushed from laptops (unsigned, unreproducible, skips checks). The tag triggers the publish workflow (`github-actions-authoring` example). Tags are immutable: a broken release gets a new patch version, never a moved tag.

6. **Changelog: keep-a-changelog format, generated then edited.** Sections Added/Changed/Fixed/Removed/Deprecated/Security, newest first, each entry linking the PR. Auto-generation from commits gives completeness; a 2-minute human edit gives sense (merge the six "fix typo" entries, translate internals into consumer language).

7. **Pre-releases and deprecation flow:** `-rc.1`/`-beta.1` suffixes for testing releases (semver orders them correctly); deprecate in a minor (warn, document the replacement), remove in the next major, and state the support window for old majors ("previous major gets security fixes for 12 months") — the removal calendar is part of the release process, not a future argument.

8. **0.x policy, explicit.** Under semver, 0.x allows breaking minors — but consumers rarely read that memo. Either commit to 1.0 early and honor it, or state loudly in the README that 0.x minors may break. Perpetual 0.x on a widely-used library is a versioning bug.

## Common pitfalls
- Breaking changes shipped as minors "because it's small." Size is irrelevant; one renamed parameter breaks every caller. Consumer impact only.
- Version bumped in three files by hand, one forgotten — `package.json` says 2.3.0, the CLI reports 2.2.0. Single source of truth, propagated by tooling.
- Changelog written from `git log` verbatim: 40 entries of "wip", "fix", "address review". Step 6's human pass exists for this.
- Marketing-driven jumps (2.4 → 5.0 to match the product line) destroying the semantic signal. Separate the brand version from the artifact version if marketing needs one.
- Monorepo packages released in lockstep so one package's fix bumps twelve unchanged packages. Independent versioning via changesets unless packages are genuinely coupled.

## Example
CLI tool at 0.9.x for two years, breaking users each minor. Applied: `VERSIONING.md` defining the contract (flags + exit codes + machine-readable `--json` output covered; human-readable output not), cut 1.0.0, changesets + release-please wired so PRs declare impact, CI-created signed tags triggering publish. Three months later a PR renamed `--out` to `--output`: changeset forced the "breaking?" question at review time → shipped as 2.0.0 with `--out` kept as deprecated alias for one major. Zero surprise-breakage issues since; changelog edits take ~3 minutes per release.

## Related skills
- `release-notes` — the human-facing narrative on top of step 6's changelog.
- `github-actions-authoring` — the tag-triggered publish workflow.
- `api-design-rest` — versioning the HTTP contract specifically.
- `dependency-upgrade` — this skill seen from the consumer's side.
