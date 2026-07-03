---
name: refactor-safely
description: >
  Use when restructuring existing code without changing behavior — extracting
  functions, renaming, moving modules, untangling dependencies — and you need
  the change to be provably safe. Triggers: "refactor this", "clean up this
  module", "extract this into", "restructure without breaking",
  "this function is too big".
---

# Refactor Safely

## When to use this skill
- Restructuring working code: extract, inline, rename, move, split.
- Preparing messy code for a feature change ("make the change easy, then make the easy change").
- NOT for rewrites that intentionally change behavior — that's feature work and needs its own spec and tests.

## Prerequisites
- Tests covering the code you're touching. If there are none, write characterization tests first (see `legacy-code-first-contact`) — refactoring untested code is guessing.
- A clean working tree. Refactoring mixed into feature diffs is unreviewable.

## Workflow

1. **Define the invariant.** Write down, in one line, what must not change: observable behavior, public API, wire format, performance envelope. Everything you do gets checked against this.

2. **Run the test suite and record the baseline.** Failing tests before you start? Fix or quarantine them first — otherwise you can't tell your breakage from pre-existing breakage.

3. **Slice into mechanical steps.** Each step is one named refactoring (extract function, rename, move file, inline variable) that compiles and passes tests on its own. If a step needs the word "and" to describe, split it.

4. **Prefer tool-assisted transforms.** IDE rename/extract, `codemod`, language-server refactors — these preserve semantics mechanically. Hand-editing 30 call sites is where typos become bugs.

5. **Commit after every green step.** `refactor: extract validate_order from process_order` — small commits make `git bisect` useful and let you abandon a bad step cheaply with `git reset` instead of untangling it.

6. **Watch for behavior leaks at the seams.** The classic ones: evaluation order changes when you extract expressions with side effects; default-argument evaluation moves; exception types change when you wrap calls; lazy becomes eager (or vice versa) when you move code across an `if`.

7. **Final check against the invariant.** Full suite, plus whatever the invariant demands beyond tests: diff the public API surface, compare serialized output on fixed inputs, benchmark if the envelope was performance.

## Common pitfalls
- "While I'm here" fixes. The moment you fix a bug mid-refactor, the diff is no longer behavior-preserving and the reviewer can't skim it. Note the bug, finish the refactor, fix it in the next commit.
- Refactoring toward an abstraction you *predict* you'll need. Refactor toward the change you're actually about to make.
- Big-bang branch that diverges for a week. Long-lived refactor branches collect merge conflicts faster than they collect value; land steps incrementally.
- Trusting green tests that never exercised the moved code. Check coverage on the refactored region, not the suite pass/fail.

## Example
Task: split a 400-line `process_payment` into stages.
Steps committed separately: (1) extract `validate_card` — tests green; (2) extract `apply_fraud_rules` — one test fails: extraction changed evaluation order so fraud rules now run before currency normalization. Step reverted with `git reset --hard HEAD~1`, re-done with order preserved, green. (3) extract `settle`; (4) rename locals. Four commits, each independently revertable, invariant (identical decline/approve decisions on the test corpus) verified at the end.

## Related skills
- `legacy-code-first-contact` — getting the test safety net when none exists.
- `code-review-checklist` — reviewing someone else's refactor.
- `debugging-by-bisection` — small refactor commits are what make bisection work later.
