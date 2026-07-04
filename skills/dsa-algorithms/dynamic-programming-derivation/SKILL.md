---
name: dynamic-programming-derivation
description: >
  Use when a problem needs dynamic programming and you want the derivation
  path — recognize overlapping subproblems, define the state, write the
  recurrence, memoize, then tabulate — instead of pattern-matching to a
  memorized solution. Triggers: "is this DP", "derive the recurrence",
  "minimum cost to reach", "number of ways to", "longest common",
  "knapsack-like". NOT when a greedy exchange argument works, and NOT
  when subproblems don't overlap (plain divide & conquer).
---

# Dynamic Programming Derivation

## Overview

DP is a derivation, not a catalog: confirm the two signatures, write the state as a complete sentence, derive the recurrence from the last decision, and validate against brute force. The state sentence is 80% of the work — pattern-matching to a memorized LeetCode solution whose invariant doesn't hold here is the genre's signature failure.

## When to Use

- Counting/optimization over sequences, choices, or partitions where brute force branches exponentially.
- Interview prep — the derivation method transfers where memorized solutions don't.

**When NOT to use:**
- A greedy exchange argument works (locally-best provably never hurts) — DP is 10× the code for the same answer.
- Subproblems don't overlap — plain divide & conquer.

## Prerequisites

- Recursion fluency and `complexity-analysis` step 4 (distinct states × work per state — the bound every DP lives by).

## The Workflow

1. **Confirm the two signatures before committing:** (a) **optimal substructure** — the best whole answer composes from best sub-answers; (b) **overlapping subproblems** — the naive recursion revisits the same instance. Quick test: write the brute-force recursion signature and count *distinct* argument tuples. Polynomial count + exponential tree = DP. Exponential distinct states = DP won't save you; rethink the state or the problem.

2. **Define the state as a complete sentence — this is 80% of the work.** Template: `dp[args] = <the best/count of X, considering exactly Y, with Z decided>`. The args must capture *everything the future needs to know about the past* — no more, no less. "dp[i] = max profit using the first i items" is incomplete for knapsack (the future needs remaining capacity) → `dp[i][c]`. When the recurrence needs information the state doesn't carry, the state is wrong — add the dimension, don't patch with globals.

3. **Write the recurrence as "the last decision":** for state S, enumerate the possible final choices, each reducing to a smaller state: `dp[i][c] = max(dp[i-1][c], value[i] + dp[i-1][c-weight[i]])` — skip item i, or take it. Then pin the **base cases** and the **invalid states** (negative capacity → −∞ for max problems, so invalid paths lose automatically) — 0 where −∞ belonged makes impossibility look free, and max() happily routes through it.

4. **Implement top-down memoized first — it's the recurrence, verbatim:**
   ```python
   @functools.cache
   def best(i, cap):
       if i < 0 or cap == 0: return 0
       if weight[i] > cap: return best(i-1, cap)
       return max(best(i-1, cap), value[i] + best(i-1, cap - weight[i]))
   ```
   Memoized top-down is easier to get right and is a legitimate final answer. Convert to bottom-up when: recursion depth threatens the stack, constants matter, or space optimization (step 5) is needed.

5. **Tabulate by topological order of dependencies, then shrink space:** the loop order must compute every state after the states it reads (prefix DPs: increasing i; interval DPs: increasing LENGTH — the one everyone gets wrong first try). If `dp[i][*]` reads only `dp[i-1][*]`, keep two rows — or one row with the direction trick (0/1 knapsack: iterate capacity DOWNWARD so each item counts once; upward gives unbounded knapsack — the direction IS the semantics, comment it).

6. **State the complexity and sanity-check it:** states × transition cost. n × W × O(1) = O(nW) — and note when that's *pseudo-polynomial* (W is a magnitude, not an input length; W=10¹² means this DP is dead and the problem wants different modeling). State count over ~10⁷–10⁸ → redesign the state before optimizing constants.

7. **Recover the answer if asked, and validate against brute force.** Store argmax choices or walk backward re-checking transitions. Then the five-minute oracle: a DP that disagrees with the n=8 exhaustive answer has a wrong state or base case (`property-based-testing`'s oracle pattern, applied inward). This check is not optional — it's the cheapest correctness proof DP admits.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "This is basically knapsack — I know that solution" | 'Basically' is where the invariant differs. If you can't write THIS problem's state sentence, you don't have the problem yet — the memorized solution's assumptions ride in silently. |
| "My state is close enough — I'll patch the missing info with a global" | A state that needs external memory isn't a state; the memo keys become wrong and cache hits return answers to different questions. Add the dimension. |
| "Base case 0 everywhere — simpler" | 0 where −∞ belonged makes invalid paths look free, and the max routes through impossibility. Sentinel semantics are correctness, not style. |
| "Brute-force validation is beneath this — the logic is clearly right" | Wrong fill orders and incomplete states produce plausible numbers, not crashes. The n=8 oracle takes five minutes and catches what inspection can't (the example's off-by-one). |
| "Tabulation is the real DP — skip the memo version" | Top-down IS the recurrence verbatim, visits only reachable states, and is a legitimate final answer. Bottom-up earns its keep for depth/space reasons, not purity. |
| "O(nW) is polynomial — ship it" | W=10¹² says otherwise: pseudo-polynomial bounds are exponential in input LENGTH. Step 6's magnitude check is what stands between the analysis and the timeout. |

## Red Flags

- No state sentence anywhere — code written straight from a remembered pattern.
- Cache keyed on mutable state, or on args that don't affect the result.
- Interval DP iterated by index instead of by length.
- One-row knapsack iterating capacity upward for a 0/1 problem (silent unbounded semantics).
- No brute-force comparison at small n.
- "DP" running at brute-force speed (memo never hitting — key bug).

## Verification

- [ ] Both signatures confirmed: distinct-state count stated vs the naive tree — one line.
- [ ] State sentence written in full — in a comment above the function.
- [ ] Base cases and invalid-state sentinels explicit, with max/min semantics matched.
- [ ] Fill order justified for tabulation (dependency direction stated); direction tricks commented.
- [ ] Complexity stated as states × transition; pseudo-polynomial flagged if W is a magnitude.
- [ ] Brute-force oracle passes at small n — test linked.

## Example

Real task: minimum-cost batching of 300 schema migrations, where running migrations i..j as one batch costs `setup + Σrisk(i..j)²`. Brute force: all partitions = 2²⁹⁹. Step 1: last decision = "the final batch starts at k" → distinct states = suffix start index = 300 — overlapping heavily. State sentence: `dp[i] = min total cost to run migrations i..n`. Recurrence: `dp[i] = min over j≥i of batchcost(i,j) + dp[j+1]`; base `dp[n] = 0`. States × transition = 300 × 300 = 9×10⁴ with O(1) batchcost via prefix sums. Implemented memoized (12 lines), validated against brute force at n=12 (caught an off-by-one in the prefix-sum bounds — the five-minute oracle paying out), answer recovery via stored split points produced the actual batch plan. Total risk cost 40% below the hand-made batching it replaced.

## Related skills

- `complexity-analysis` — the states×work bound and pseudo-polynomial warning.
- `graph-traversal-patterns` — DP on DAGs is traversal + values; cycles mean it's not a DP.
- `binary-search-variants` — "minimize the maximum" often prefers search-on-answer over DP.
- `property-based-testing` — the brute-force oracle in step 7.
