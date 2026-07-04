---
name: refactor-safely
description: >
  Use when restructuring existing code without changing behavior —
  extracting functions, renaming, moving modules, untangling dependencies —
  and the change must be provably safe. Triggers: "refactor this", "clean
  up this module", "extract this into", "restructure without breaking",
  "this function is too big". NOT for rewrites that intentionally change
  behavior (that's feature work with its own spec) and NOT for getting
  untested code under test first (see legacy-code-first-contact).
---

# Refactor Safely

## Overview

Refactoring is only refactoring if behavior provably doesn't change — otherwise it's editing with optimism. Small mechanical steps with test gates between them turn a risky restructure into a sequence of boring, revertible commits.

## When to Use

- Restructuring working code: extract, inline, rename, move, split.
- Preparing messy code for a feature change ("make the change easy, then make the easy change").

**When NOT to use:**
- Rewrites that intentionally change behavior — feature work; needs its own spec and tests.
- Code with no tests — run `legacy-code-first-contact` first; refactoring untested code is guessing.

## Prerequisites

- Tests covering the code you're touching.
- A clean working tree. Refactoring mixed into feature diffs is unreviewable.

## The Workflow

1. **Define the invariant.** Write down, in one line, what must not change: observable behavior, public API, wire format, performance envelope. Everything you do gets checked against this.

2. **Run the test suite and record the baseline.** Failing tests before you start? Fix or quarantine them first — otherwise you can't tell your breakage from pre-existing breakage.

3. **Slice into mechanical steps.** Each step is one named refactoring (extract function, rename, move file, inline variable) that compiles and passes tests on its own. If a step needs the word "and" to describe, split it.

4. **Prefer tool-assisted transforms.** IDE rename/extract, `codemod`, language-server refactors — these preserve semantics mechanically. Hand-editing 30 call sites is where typos become bugs.

5. **Commit after every green step.** `refactor: extract validate_order from process_order` — small commits make `git bisect` useful and let you abandon a bad step cheaply with `git reset` instead of untangling it.

6. **Watch for behavior leaks at the seams.** The classic ones: evaluation order changes when you extract expressions with side effects; default-argument evaluation moves; exception types change when you wrap calls; lazy becomes eager (or vice versa) when you move code across an `if`.

7. **Final check against the invariant.** Full suite, plus whatever the invariant demands beyond tests: diff the public API surface, compare serialized output on fixed inputs, benchmark if the envelope was performance. Warning: green tests that never exercised the moved code prove nothing — check coverage on the refactored region, not the suite's pass/fail.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "While I'm here, I'll fix this little bug too" | The moment a bug fix enters, the diff is no longer behavior-preserving and the reviewer can't skim it as safe. Note the bug, finish the refactor, fix it in the next commit. |
| "These steps are so small, committing each one is ceremony" | The commits are the rollback mechanism. One tangled commit means one bad step costs the whole refactor; per-step commits make abandonment a `git reset`. |
| "I'll refactor toward the design we'll need next quarter" | Speculative structure is the thing refactoring exists to remove. Refactor toward the change you're actually about to make. |
| "Tests are green, so the extraction was safe" | Green proves the tested paths held. The seam leaks (evaluation order, exception types, lazy/eager) hide precisely where coverage is thin — step 7's coverage check exists for this. |
| "A long-lived branch keeps the refactor reviewable in one piece" | Refactor branches collect merge conflicts faster than value. Land steps incrementally; the reviewable unit is the step, not the saga. |
| "Hand-editing the call sites is faster than setting up the codemod" | For 5 sites, true. For 30, the one typo you'll make costs more than the tool setup — and mechanical transforms don't get tired at site 24. |

## Red Flags

- The "refactor" PR's diff contains logic changes, new features, or bug fixes.
- One commit titled "refactor everything" touching 40 files.
- Tests were failing before the refactor started and are "still just those ones" failing after.
- The invariant was never written down — asked what must not change, the author says "the code should work."
- Coverage on the moved region unknown; confidence sourced from suite-green alone.
- Merge conflicts accumulating on a week-old refactor branch.

## Verification

- [ ] Invariant written (one line) — in the PR description.
- [ ] Baseline suite run recorded before step one (link or output).
- [ ] Each step is a separate green commit with a named refactoring in its message — history linked.
- [ ] Coverage on the refactored region checked, not just suite pass/fail — number or report attached.
- [ ] Invariant-specific final check done (API diff, output comparison, or benchmark as applicable) — artifact attached.
- [ ] Zero behavior changes in the diff — reviewer can confirm by skimming step commits.

## Example

Task: split a 400-line `process_payment` into stages.
Steps committed separately: (1) extract `validate_card` — tests green; (2) extract `apply_fraud_rules` — one test fails: extraction changed evaluation order so fraud rules now run before currency normalization. Step reverted with `git reset --hard HEAD~1`, re-done with order preserved, green. (3) extract `settle`; (4) rename locals. Four commits, each independently revertable, invariant (identical decline/approve decisions on the test corpus) verified at the end.

## Related skills

- `legacy-code-first-contact` — getting the test safety net when none exists.
- `code-review-checklist` — reviewing someone else's refactor.
- `debugging-by-bisection` — small refactor commits are what make bisection work later.
