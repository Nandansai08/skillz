---
name: github-actions-authoring
description: >
  Use when writing or debugging GitHub Actions workflows — triggers,
  matrices, secrets handling, reusable workflows, permissions, fork-PR
  safety. Triggers: "write a GitHub Action", "workflow yaml", "actions
  matrix", "why didn't my workflow trigger", "reusable workflow",
  "workflow_dispatch". NOT for pipeline architecture decisions — what
  runs when lives in ci-pipeline-design; this is the Actions-specific
  mechanics and security.
---

# GitHub Actions Authoring

## Overview

Workflows are executable code holding credentials, triggered by partially attacker-controlled events — authoring them is equal parts syntax and security. Precise triggers, minimum permissions, SHA-pinned actions, and injection-safe expressions are the floor, not the polish.

## When to Use

- Writing a new workflow or debugging trigger/permission surprises.
- Reviewing workflow changes (review them like code with credentials — they are).

**When NOT to use:**
- Deciding pipeline shape and stage routing — `ci-pipeline-design`.

## Prerequisites

- Repo access to `.github/workflows/`; `act` or a scratch branch for iteration.

## The Workflow

1. **Choose triggers precisely — the common table:**
   - `pull_request` — safe default for CI; runs against the *merge* ref with read-only token on fork PRs.
   - `push: branches: [main]` — post-merge jobs (deploy, publish).
   - `pull_request_target` — DANGER: runs *base* branch code with write token + secrets against attacker-controlled PR context. Only for label/comment automation; never checkout-and-execute PR code under it.
   - `workflow_dispatch` — manual runs with typed inputs; add to anything you'll ever need to re-run.
   - `schedule` — cron in UTC, runs from the default branch only; jobs on quiet repos get disabled after 60 days.

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

6. **Reuse via reusable workflows (`workflow_call`) for job-level patterns** shared across repos; composite actions for step-level snippets. Version them like code (tags), and pass secrets explicitly (`secrets: inherit` only inside a trusted org). Warning: `secrets` isn't available in `if:` conditions — gate via a step that checks and sets an output.

7. **Debug in order:** the workflow didn't trigger → check event/branch/path filters and whether the workflow file exists on the *default* branch (most triggers read it from there); step behaves oddly → `ACTIONS_STEP_DEBUG=true` repo secret for verbose logs; context confusion → dump it once: `run: echo '${{ toJSON(github.event) }}'`. Iterate on a scratch branch with `workflow_dispatch` instead of pushing to main.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "pull_request_target lets the bot work on fork PRs — convenient" | It's the canonical self-pwn: attacker-controlled code meets your write token and secrets. If you must use it, never execute checked-out PR content — the convenience is the vulnerability. |
| "SHA-pinning is unreadable; tags are fine for popular actions" | Popular is why they're targeted — the 2025 tag-rewrite attack hit one of the most-used actions on the platform. The `# v6.9` comment restores readability; Dependabot maintains the pins. |
| "Our repo is private, injection doesn't apply" | Branch names and PR titles are written by anyone with repo access today and by the contractor/integration of tomorrow. Env-var indirection costs two lines; the habit is the control. |
| "Default token permissions are fine — we trust our steps" | You're trusting every transitive action author too. The one-line permissions block is the cheapest blast-radius cut in the platform. |
| "No timeout needed, the tests are fast" | The default is six HOURS. One hung test burns a workday of runner quota per occurrence; `timeout-minutes` is one line. |
| "I'll test the workflow by merging it" | Trigger surprises are the norm (default-branch rule, filter mismatches). The scratch-branch + workflow_dispatch loop finds them without polluting main's history. |

## Red Flags

- `pull_request_target` + `actions/checkout` of PR head in the same workflow.
- Third-party actions at mutable tags in workflows holding secrets.
- No `permissions:` block; org default is write-all.
- `${{ github.event... }}` interpolated directly inside `run:` strings.
- Jobs without `timeout-minutes`.
- Cron workflows that silently stopped (60-day disable) with nobody noticing.

## Verification

- [ ] `permissions:` block present and minimal in every workflow touched — diff shows it.
- [ ] All third-party actions SHA-pinned with version comments — grep output attached.
- [ ] No untrusted context in `run:` interpolations — grep for `\$\{\{.*github\.event` inside run blocks, hits justified.
- [ ] Every job has `timeout-minutes`; concurrency group set for PR workflows.
- [ ] Fork-PR behavior stated: which jobs run, what token they get — one sentence in the PR.
- [ ] Workflow exercised via scratch branch or `workflow_dispatch` before merge — run linked.

## Example

Request: "publish the package when a release is tagged." Delivered workflow: `on: push: tags: ['v*']`, `permissions: {contents: read, id-token: write}` for npm provenance via OIDC (no long-lived NPM_TOKEN secret), SHA-pinned actions, `timeout-minutes: 10`, and a `workflow_dispatch` input for dry-run republish. Review caught one issue: the build step interpolated the tag name into `run:` unquoted — moved to env var per step 5, since tag names are attacker-controlled in forks.

## Related skills

- `ci-pipeline-design` — the architecture this syntax implements.
- `secrets-management` — OIDC-over-static-secrets rationale and rotation.
- `release-versioning` — the tagging scheme that triggers the example.
