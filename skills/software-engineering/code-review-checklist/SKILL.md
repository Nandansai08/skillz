---
name: code-review-checklist
description: >
  Use when reviewing a pull request, diff, or code change and you want a
  systematic pass instead of ad-hoc skimming. Orders review by severity:
  correctness first, then security, performance, style last. Triggers:
  "review this PR", "review my diff", "look over this change",
  "is this ready to merge".
---

# Code Review Checklist

## When to use this skill
- Reviewing a PR, branch diff, or pasted code change of any size.
- Doing a self-review before opening a PR.
- NOT for whole-repo audits — that's an architecture or tech-debt review, different scope.

## Prerequisites
- The diff plus enough surrounding context to see how changed functions are called. Review the change *in context*, never the diff alone.

## Workflow

1. **Read the intent first.** PR description, linked ticket, commit messages. If you can't state in one sentence what the change is supposed to do, ask before reviewing — you can't judge correctness against an unknown spec.

2. **Correctness pass (highest severity).** For each changed function:
   - Trace one happy path and one failure path by hand.
   - Check every boundary the diff touches: off-by-one, empty collection, null/None, zero, negative, max size.
   - Grep for other callers of any function whose signature or behavior changed — the diff shows the change, not the blast radius.
   - Concurrency: shared state touched without synchronization? Check-then-act races?

3. **Security pass.** Only where the diff crosses a trust boundary: user input reaching queries/commands/paths (injection), authz checks on new endpoints, secrets in code or logs, unsafe deserialization.

4. **Performance pass.** Only flag what's measurable: queries inside loops (N+1), unbounded result sets, O(n²) on collections that grow with users, missing pagination. Don't speculate about micro-optimizations.

5. **Tests pass.** Does a test exist that would fail if the core logic were reverted? If the PR fixes a bug, is there a regression test reproducing it? Missing coverage on the main path is a blocking comment; missing edge-case tests is a suggestion.

6. **Style/naming pass (lowest severity, non-blocking).** Only comment where naming actively misleads or the code diverges from established local patterns. Never block a PR on formatting a linter should catch.

7. **Write the review.** Severity-tag every comment (`blocking:` / `suggestion:` / `nit:`). Lead with the most severe. State what's wrong and what would resolve it — "this breaks when list is empty; early-return or guard" not "hmm, edge cases?"

## Common pitfalls
- Reviewing only the green/red lines. Bugs live in unchanged code that now gets called differently — always open the surrounding function.
- Fifteen nits and no correctness check. Style comments feel productive and are the least valuable thing a reviewer produces.
- Approving because tests pass. Tests verify what the author thought of; the reviewer's job is what they didn't.
- Asking for unrelated refactors. Scope creep in review stalls delivery; file a follow-up issue instead.

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
