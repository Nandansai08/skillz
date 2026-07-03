---
name: coverage-analysis
description: >
  Use when reading a code-coverage report to decide what actually needs
  testing — branch vs line coverage, which gaps matter, when a number is
  lying. Triggers: "coverage report", "is 80% coverage enough", "what's not
  tested", "increase coverage", "coverage dropped on this PR".
---

# Coverage Analysis

## When to use this skill
- Interpreting a coverage report or a coverage gate failure on a PR.
- Deciding where to spend the next unit of testing effort.
- NOT for making tests better — coverage only shows what *ran*; see `unit-test-design` for whether tests verify anything.

## Prerequisites
- Coverage tooling wired up with **branch coverage** on (`pytest --cov --cov-branch`, `go test -covermode=count`, Istanbul/nyc, JaCoCo). Line-only coverage is the first lie to fix.

## Workflow

1. **Read branch coverage, not line coverage.** `if user.is_admin or order.total > limit:` is one line, four meaningful paths. Line coverage says 100% when one path ran. Configure branch mode before drawing any conclusion.

2. **Look at gaps, not the percentage.** Open the HTML report and read the red regions in order of risk:
   - uncovered **error/except paths** in money, auth, data-writing code — highest value gaps, they're where silent failures live;
   - uncovered branches in complex conditionals;
   - whole uncovered files that are load-bearing (vs. dead code — see step 5).
   A repo at 85% with red error paths in billing is worse-tested than one at 70% with red `__repr__`s.

3. **Distinguish "covered" from "verified."** Spot-check: pick 3 green high-risk regions, mutate the code by hand (flip a comparison, remove a guard), rerun. Suite stays green → those lines are executed-but-unasserted. Mutation tools automate this (`mutmut`, `pitest`, Stryker) — run on the critical package, not the whole repo (too slow).

4. **Use diff coverage for gates, not absolute thresholds.** Gate PRs on "changed lines ≥ 90% covered" (`diff-cover`, Codecov patch status). Absolute repo thresholds ratchet legacy debt onto every feature PR and invite gaming (tests that execute without asserting).

5. **Treat uncovered code as a deletion candidate first.** Before writing tests for a red region, check: is it reachable at all? `git log` it, grep for callers. Dead code found via coverage is a gift — delete instead of test.

6. **Exclude deliberately and visibly.** Generated code, `if TYPE_CHECKING`, `__main__` blocks, debug-only paths — exclude via config (`.coveragerc [report] exclude_lines`) with a comment, so the number describes code you mean to test. Silent exclusions are how 90% means 60%.

7. **Set targets by tier, not globally.** Working defaults: core domain logic 90%+ branch; adapters/IO glue 60–80% (integration tests carry these); UI/scripts best-effort. Write the tiers down; argue about the tiers once instead of every PR.

## Common pitfalls
- Chasing the last 10% into getters and generated code while a payment error branch sits red. Percentage optics over risk.
- Coverage collected only from unit tests when integration tests cover the adapters — the report shows false gaps. Merge coverage across suites (`coverage combine`) before judging.
- `assert result is not None` tests written to satisfy a gate. Executed, unverified — mutation check (step 3) is the antidote.
- Reading coverage as quality. It's a *necessary-not-sufficient* detector: red is definitely untested, green is merely executed.
- Gating on absolute % then watching teams exclude their way to compliance. Diff coverage + visible excludes (steps 4, 6).

## Example
Repo at "87%, healthy." Branch mode dropped it to 74%. HTML report: `refund.py` error paths fully red — every `except StripeError` branch untested. Mutation spot-check on the green `apply_credit`: removed the negative-amount guard, suite stayed green — executed by a test asserting only `status == 200`. Outcome: 6 tests on refund error paths, 1 real assert added, 300 lines of unreachable legacy exporter deleted (coverage found it), diff-coverage gate at 90% replaced the absolute gate. New number: 79% — and honest.

## Related skills
- `unit-test-design` — writing tests that make green mean verified.
- `regression-test-from-bug` — the highest-value coverage additions come from real bugs.
- `metric-definition` — coverage is a textbook gameable metric; same discipline applies.
