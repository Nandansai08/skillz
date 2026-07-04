---
name: coverage-analysis
description: >
  Use when reading a code-coverage report to decide what actually needs
  testing — branch vs line coverage, which gaps matter, when the number is
  lying. Triggers: "coverage report", "is 80% coverage enough", "what's
  not tested", "increase coverage", "coverage dropped on this PR". NOT
  for making tests verify more (see unit-test-design) — coverage only
  shows what ran, never what was checked.
---

# Coverage Analysis

## Overview

Coverage is a necessary-not-sufficient detector: red is definitely untested, green is merely executed. Reading it right means branch mode on, gaps ranked by risk, and the occasional mutation spot-check to catch executed-but-unasserted green.

## When to Use

- Interpreting a coverage report or a coverage gate failure on a PR.
- Deciding where to spend the next unit of testing effort.

**When NOT to use:**
- Improving what tests verify — `unit-test-design`; coverage can't see assertions.

## Prerequisites

- Coverage tooling wired up with **branch coverage** on (`pytest --cov --cov-branch`, `go test -covermode=count`, Istanbul/nyc, JaCoCo). Line-only coverage is the first lie to fix.

## The Workflow

1. **Read branch coverage, not line coverage.** `if user.is_admin or order.total > limit:` is one line, four meaningful paths. Line coverage says 100% when one path ran. Configure branch mode before drawing any conclusion.

2. **Look at gaps, not the percentage.** Open the HTML report and read the red regions in order of risk:
   - uncovered **error/except paths** in money, auth, data-writing code — highest value gaps, they're where silent failures live;
   - uncovered branches in complex conditionals;
   - whole uncovered files that are load-bearing (vs. dead code — see step 5).
   A repo at 85% with red error paths in billing is worse-tested than one at 70% with red `__repr__`s.

3. **Distinguish "covered" from "verified."** Spot-check: pick 3 green high-risk regions, mutate the code by hand (flip a comparison, remove a guard), rerun. Suite stays green → those lines are executed-but-unasserted. Mutation tools automate this (`mutmut`, `pitest`, Stryker) — run on the critical package, not the whole repo (too slow).

4. **Use diff coverage for gates, not absolute thresholds.** Gate PRs on "changed lines ≥ 90% covered" (`diff-cover`, Codecov patch status). Absolute repo thresholds ratchet legacy debt onto every feature PR and invite gaming (tests that execute without asserting).

5. **Treat uncovered code as a deletion candidate first.** Before writing tests for a red region, check: is it reachable at all? `git log` it, grep for callers. Dead code found via coverage is a gift — delete instead of test.

6. **Exclude deliberately and visibly.** Generated code, `if TYPE_CHECKING`, `__main__` blocks, debug-only paths — exclude via config (`.coveragerc [report] exclude_lines`) with a comment, so the number describes code you mean to test. Silent exclusions are how 90% means 60%. Warning: merge coverage across suites (`coverage combine`) before judging — unit-only reports show false gaps where integration tests carry the adapters.

7. **Set targets by tier, not globally.** Working defaults: core domain logic 90%+ branch; adapters/IO glue 60–80% (integration tests carry these); UI/scripts best-effort. Write the tiers down; argue about the tiers once instead of every PR.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "We're at 87% — the suite is healthy" | Branch mode dropped the example repo to 74%, with billing error paths fully red. The aggregate number hides exactly the gaps that matter; only the gap-read (step 2) knows. |
| "Line coverage is close enough to branch coverage" | One line, four paths. Compound conditionals are where logic bugs live, and line mode scores them covered at 25% exercised. |
| "The gate requires 80%, so I'll add tests until it passes" | Gate-chasing produces `assert result is not None` tests — executed, unverified, and permanent maintenance load. Diff coverage on changed lines (step 4) aims the effort; mutation checks (step 3) audit it. |
| "Green means tested" | Green means executed. The mutation spot-check takes ten minutes and routinely finds guards whose removal changes nothing in the suite. |
| "That uncovered file must need tests — add them" | Or it's dead. The git-log-and-callers check first: deleting unreachable code beats testing it, permanently. |
| "Excluding generated code inflates our honesty problem" | Unexcluded generated code DEFLATES signal — the number stops describing decisions anyone can act on. Visible, commented exclusions are the honest configuration. |

## Red Flags

- Coverage discussed as a single number, never as specific red regions.
- Line-only mode in the CI config.
- Tests landing in the same PR as a gate failure, all assertion-light.
- Exclusion config with no comments, growing quarterly.
- Unit-suite-only coverage judging a repo whose adapters are integration-tested.
- 100% covered modules whose deliberate breakage fails nothing (never mutation-checked).

## Verification

- [ ] Branch mode confirmed in the CI config — link.
- [ ] Gap analysis produced: top red regions listed by risk, each dispositioned (test/delete/accept) — in the PR or report.
- [ ] Mutation spot-check run on ≥3 green high-risk regions — results noted (survived mutants = findings).
- [ ] Gates are diff-based; absolute thresholds absent or justified — config linked.
- [ ] Exclusions all commented; coverage merged across suites before judgment.
- [ ] Tier targets documented once — linked, not re-argued in the PR.

## Example

Repo at "87%, healthy." Branch mode dropped it to 74%. HTML report: `refund.py` error paths fully red — every `except StripeError` branch untested. Mutation spot-check on the green `apply_credit`: removed the negative-amount guard, suite stayed green — executed by a test asserting only `status == 200`. Outcome: 6 tests on refund error paths, 1 real assert added, 300 lines of unreachable legacy exporter deleted (coverage found it), diff-coverage gate at 90% replaced the absolute gate. New number: 79% — and honest.

## Related skills

- `unit-test-design` — writing tests that make green mean verified.
- `regression-test-from-bug` — the highest-value coverage additions come from real bugs.
- `metric-definition` — coverage is a textbook gameable metric; same discipline applies.
