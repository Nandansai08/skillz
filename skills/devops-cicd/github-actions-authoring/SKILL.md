---
name: github-actions-authoring
description: >
  Use when writing or debugging GitHub Actions workflows — triggers, matrices,
  secrets handling, reusable workflows, permissions, fork-PR safety.
  Triggers: "write a GitHub Action", "workflow yaml", "actions matrix",
  "why didn't my workflow trigger", "reusable workflow", "workflow_dispatch".
---

# GitHub Actions Authoring

## When to use this skill
- Writing a new workflow or debugging trigger/permission surprises.
- Reviewing workflow changes (they're executable code with credentials — review them like it).
- NOT for pipeline architecture decisions (what runs when) — that's `ci-pipeline-design`; this is the Actions-specific mechanics.

## Prerequisites
- Repo access to `.github/workflows/`; `act` or a scratch branch for iteration.

## Workflow

1. **Choose triggers precisely — the common table:**
   - `pull_request` — safe default for CI; runs against the *merge* ref with read-only token on fork PRs.
   - `push: branches: [main]` — post-merge jobs (deploy, publish).
   - `pull_request_target` — DANGER: runs *base* branch code with write token + secrets against attacker-controlled PR context. Only for label/comment automation, never checking out PR code without extreme care.
   - `workflow_dispatch` — manual runs with typed inputs; add to anything you'll ever need to re-run.
   - `schedule` — cron in UTC; jobs on quiet repos get disabled after 60 days, and busy times (top of hour) queue long.

2. **Set permissions to the minimum, explicitly, per workflow:**
   ```yaml
   permissions:
     contents: read        # default-deny; add back only what's used
   ```
   The default token in many orgs is write-all. A `permissions:` block at workflow top is one line and cuts the blast radius of any compromised step.

3. **Pin third-party actions to a full SHA, not a tag:**
   ```yaml
   uses: docker/build-push-action@2cdde995de11960a780f60046be8a76e01885b57 # v6.9
   ```
   Tags are mutable — the `tj-actions/changed-files` compromise (2025) rewrote existing tags to exfiltrate secrets. First-party `actions/*` at major tag is acceptable; everything else, SHA-pin (Dependabot keeps the pins updated).

4. **Structure with the standard skeleton:** `concurrency` to cancel superseded runs, `timeout-minutes` on every job (default is 6 *hours*), caching keyed on lockfiles:
   ```yaml
   concurrency:
     group: ${{ github.workflow }}-${{ github.ref }}
     cancel-in-progress: true
   jobs:
     test:
       runs-on: ubuntu-latest
       timeout-minutes: 15
       strategy:
         fail-fast: false            # see all matrix failures, not just first
         matrix: { node: [20, 22] }
   ```

5. **Handle expressions and injection.** Never interpolate untrusted input (`github.event.pull_request.title`, branch names, comments) directly into `run:` — it's shell injection:
   ```yaml
   env:
     PR_TITLE: ${{ github.event.pull_request.title }}   # env var, then
   run: echo "$PR_TITLE"                                 # quoted use — safe
   ```

6. **Reuse via reusable workflows (`workflow_call`) for job-level patterns** shared across repos; composite actions for step-level snippets. Version them like code (tags), and pass secrets explicitly (`secrets: inherit` only inside a trusted org).

7. **Debug in order:** the workflow didn't trigger → check event/branch/path filters and whether the workflow file exists on the *default* branch (most triggers read it from there); step behaves oddly → `ACTIONS_STEP_DEBUG=true` repo secret for verbose logs; context confusion → dump it once: `run: echo '${{ toJSON(github.event) }}'`. Iterate on a scratch branch with `workflow_dispatch` instead of pushing to main.

## Common pitfalls
- `pull_request_target` + `actions/checkout` of the PR head: the canonical self-pwn — attacker code runs with your secrets. If you must, checkout to a subdir, never execute it.
- Forgetting cron is UTC and workflows-on-schedule only run from the default branch.
- Matrix + `fail-fast` default cancelling siblings, so one flaky shard hides the real failure signal.
- `secrets` in `if:` conditions — secrets aren't available there; gate via a step that checks and sets an output.
- No `timeout-minutes`: a hung test burns 6 hours of runner quota per occurrence.
- Editing a workflow and testing it via PR when the trigger only fires from main — the "why isn't it running" classic (step 7).

## Example
Request: "publish the package when a release is tagged." Delivered workflow: `on: push: tags: ['v*']`, `permissions: {contents: read, id-token: write}` for npm provenance via OIDC (no long-lived NPM_TOKEN secret), SHA-pinned actions, `timeout-minutes: 10`, and a `workflow_dispatch` input for dry-run republish. Review caught one issue: the build step interpolated the tag name into `run:` unquoted — moved to env var per step 5, since tag names are attacker-controlled in forks.

## Related skills
- `ci-pipeline-design` — the architecture this syntax implements.
- `secrets-management` — OIDC-over-static-secrets rationale and rotation.
- `release-versioning` — the tagging scheme that triggers the example.
