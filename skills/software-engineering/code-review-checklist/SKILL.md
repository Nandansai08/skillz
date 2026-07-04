---
name: code-review-checklist
description: >
  Use when reviewing a pull request, diff, or code change and you want a
  systematic pass instead of ad-hoc skimming. Triggers: "review this PR",
  "review my diff", "look over this change", "is this ready to merge",
  self-review before opening a PR. NOT for whole-repo audits (architecture
  or tech-debt scope) and NOT for the security-only deep pass (see
  secure-code-review, which this checklist invokes as one step).
---

# Code Review Checklist

## Overview

A review pass ordered by severity — correctness first, security, performance, style last — so reviewer attention lands where bugs cost the most. Ad-hoc skimming produces fifteen nits and a missed data-loss bug; the ordering is the fix.

## When to Use

- Reviewing a PR, branch diff, or pasted code change of any size.
- Doing a self-review before opening a PR.

**When NOT to use:**
- Whole-repo health audits — different scope and cadence.
- Pure security review of a sensitive surface — go directly to `secure-code-review` at full depth.

## Prerequisites

- The diff plus enough surrounding context to see how changed functions are called. Review the change *in context*, never the diff alone.

## The Workflow

1. **Read the intent first.** PR description, linked ticket, commit messages. If you can't state in one sentence what the change is supposed to do, ask before reviewing — you can't judge correctness against an unknown spec.

2. **Correctness pass (highest severity).** For each changed function:
   - Trace one happy path and one failure path by hand.
   - Check every boundary the diff touches: off-by-one, empty collection, null/None, zero, negative, max size.
   - Grep for other callers of any function whose signature or behavior changed — the diff shows the change, not the blast radius. Bugs live in unchanged code that now gets called differently; open the surrounding function, not just the green/red lines.
   - Concurrency: shared state touched without synchronization? Check-then-act races?

3. **Security pass.** Only where the diff crosses a trust boundary: user input reaching queries/commands/paths (injection), authz checks on new endpoints, secrets in code or logs, unsafe deserialization. Escalate to `secure-code-review` when the surface is auth, money, or user input at any scale.

4. **Performance pass.** Only flag what's measurable: queries inside loops (N+1), unbounded result sets, O(n²) on collections that grow with users, missing pagination. Don't speculate about micro-optimizations.

5. **Tests pass.** Does a test exist that would fail if the core logic were reverted? If the PR fixes a bug, is there a regression test reproducing it (`regression-test-from-bug`)? Missing coverage on the main path is a blocking comment; missing edge-case tests is a suggestion.

6. **Style/naming pass (lowest severity, non-blocking).** Only comment where naming actively misleads or the code diverges from established local patterns. Never block a PR on formatting a linter should catch.

7. **Write the review.** Severity-tag every comment (`blocking:` / `suggestion:` / `nit:`). Lead with the most severe. State what's wrong and what would resolve it — "this breaks when list is empty; early-return or guard" not "hmm, edge cases?" Keep scope: unrelated refactor requests become follow-up issues, not review demands.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "Tests pass, so it's probably fine" | Tests verify what the author thought of; the reviewer's entire job is what they didn't. A green suite says nothing about the empty-list path nobody tested. |
| "It's a small diff, quick skim is enough" | Small diffs to shared functions have the largest blast radii. The caller-grep (step 2) costs two minutes and is exactly what skims skip. |
| "The author is senior, they know what they're doing" | Seniority doesn't prevent boundary bugs; it prevents obvious ones. Review effort scales with blast radius, not inversely with author rank. |
| "I'll just leave style comments — the logic looks complicated" | Fifteen nits and no correctness check is the least valuable review output. If the logic is too complicated to review, that IS the blocking comment. |
| "Asking about intent will look like I didn't read it" | Reviewing against a guessed spec approves the wrong thing politely. The one-sentence intent question is the cheapest defect filter in the whole pass. |
| "Someone else will catch the security angle" | Nobody downstream re-reviews merged code. The step-3 pass is minutes; the injection it catches is an incident. |

## Red Flags

- Approval posted minutes after a non-trivial PR opened — no caller-grep happened.
- Review consists entirely of naming/formatting comments on a logic-heavy diff.
- No comment on a PR that changes a shared function's behavior.
- "LGTM, tests pass" as the entire review body.
- Comments without severity labels — the author can't tell blockers from preferences.
- The same reviewer approving their own pattern being copied unexamined across the codebase.

## Verification

- [ ] Intent stated: you can write one sentence of what the change does — and it matches the ticket.
- [ ] Caller-grep run for every changed signature/behavior — search and hit count noted in the review if non-trivial.
- [ ] Each blocking comment names the failing input/state and what would resolve it.
- [ ] Test question answered explicitly: which test fails if the core logic reverts? (Named in review or flagged as missing.)
- [ ] Every comment severity-tagged; most severe leads.

## Example

Diff adds `get_user_orders(user_id, limit=None)`. Checklist catches:
- **blocking:** `limit=None` produces an unbounded `SELECT` — existing callers pass no limit; production table has 40M rows.
- **suggestion:** new endpoint lacks a test for the 404 path (unknown user).
- **nit:** `tmp` → `orders_page` for readability.
Review posted with the unbounded query first; style nit last, marked non-blocking.

## Related skills

- `secure-code-review` — deep version of step 3 with per-language grep patterns.
- `regression-test-from-bug` — what to demand in step 5 when the PR is a bug fix.
- `refactor-safely` — when review reveals the change needs restructuring first.
