---
name: dynamic-programming-derivation
description: >
  Use when a problem needs dynamic programming and you want the derivation
  path — recognize overlapping subproblems, define the state, write the
  recurrence, memoize, then tabulate — instead of pattern-matching to a
  memorized solution. Triggers: "is this DP", "derive the recurrence",
  "minimum cost to reach", "number of ways to", "longest common", "knapsack-like".
---

# Dynamic Programming Derivation

## When to use this skill
- A counting/optimization problem over sequences, choices, or partitions where brute force branches exponentially.
- Interview prep — the derivation method transfers where memorized solutions don't.
- NOT when a greedy exchange argument works (if a locally-best choice provably never hurts, DP is overkill) or when subproblems don't overlap (that's plain divide & conquer).

## Prerequisites
- Recursion fluency and `complexity-analysis` step 4 (distinct states × work per state — the bound every DP lives by).

## Workflow

1. **Confirm the two signatures before committing:** (a) **optimal substructure** — the best answer for the whole is composable from best answers to smaller instances; (b) **overlapping subproblems** — the naive recursion tree revisits the same instance many times. Quick test: write the brute-force recursion signature and ask "how many *distinct* argument tuples can occur?" Polynomial count + exponential tree = DP. Exponential distinct states = DP won't save you (rethink the state or the problem).

2. **Define the state as a complete sentence — this is 80% of the work.** Template: `dp[args] = <the best/count of X, considering exactly Y, with Z decided>`. The args must capture *everything the future needs to know about the past* — no more, no less. "dp[i] = max profit using the first i items" is incomplete for knapsack (the future needs remaining capacity) → `dp[i][c] = max profit from the first i items with capacity c`. When the recurrence in step 3 needs information the state doesn't carry, the state is wrong — add the missing dimension, don't patch with globals.

3. **Write the recurrence as "the last decision":** for state S, enumerate the possible final choices, each reducing to a smaller state: `dp[i][c] = max(dp[i-1][c], value[i] + dp[i-1][c-weight[i]])` — skip item i, or take it. Then pin the **base cases** (empty prefix, zero capacity) and the **invalid states** (negative capacity → −∞ for max problems, so invalid paths lose automatically; the sentinel choice here is a classic bug source — 0 vs −∞ vs None mean different things).

4. **Implement top-down memoized first — it's the recurrence, verbatim:**
   ```python
   @functools.cache
   def best(i, cap):
       if i < 0 or cap == 0: return 0
       if weight[i] > cap: return best(i-1, cap)
       return max(best(i-1, cap), value[i] + best(i-1, cap - weight[i]))
   ```
   Memoized top-down is easier to get right (visits only reachable states, base cases explicit) and is a legitimate final answer. Convert to bottom-up tabulation when: recursion depth threatens the stack (`complexity-analysis` step 6), the constant factor matters, or space optimization (step 5) is needed.

5. **Tabulate by topological order of dependencies, then shrink space:** the loop order must compute every state after the states it reads (for prefix-DPs: increasing i; interval DPs: increasing interval length — the one everyone gets wrong first try). If `dp[i][*]` reads only `dp[i-1][*]`, keep two rows — or one row with the *iteration direction trick* (0/1 knapsack: iterate capacity DOWNWARD so each item is counted once; upward gives unbounded knapsack — the direction IS the semantics, comment it).

6. **State the complexity and sanity-check it:** states × transition cost. n items × W capacities × O(1) transition = O(nW) — and note when that's *pseudo-polynomial* (W is a number's magnitude, not an input length; W=10¹² means this DP is dead and the problem wants meet-in-the-middle or different modeling). If the state count exceeds ~10⁷–10⁸, redesign the state before optimizing constants.

7. **Recover the answer, not just its value, if asked:** either store the argmax choice per state during the fill, or walk backward from the final state re-checking which transition was taken. And validate against brute force on tiny inputs — a DP that disagrees with the n=8 exhaustive answer has a wrong state or base case, and this check takes five minutes (`property-based-testing`'s oracle pattern, applied inward).

## Common pitfalls
- Pattern-matching to a memorized LeetCode solution whose invariant doesn't hold here — the "it's basically knapsack" that isn't. The step-2 sentence discipline is the antidote: if you can't write the state sentence, you don't have the problem yet.
- Incomplete state (the future needed something the past didn't record) — symptoms: the recurrence wants to "peek" at choices already made, or small-input validation fails only on specific orderings.
- Base-case sentinel confusion: 0 where −∞ belonged makes invalid paths look free; the max() then happily routes through impossibility.
- Wrong fill order in tabulation — reading cells not yet computed. Interval DP (matrix chain, palindrome partitioning) by index instead of by length is the canonical instance.
- The 0/1-vs-unbounded direction bug in the one-row optimization (step 5) — silent, plausible outputs, wrong answers only when an item *would* be reused.
- Memoizing on mutable/unhashable state, or including an arg in the cache key that doesn't affect the result (cache hit rate craters, "DP" runs at brute-force speed).
- Reaching for DP when greedy provably works (activity selection, fractional knapsack) — 10× the code for the same answer.

## Example
Real task: minimum-cost batching of 300 schema migrations, where running migrations i..j as one batch costs `setup + Σrisk(i..j)²` (quadratic risk penalty discourages huge batches). Brute force: all partitions = 2²⁹⁹. Step 1: last decision = "the final batch starts at k" → distinct states = suffix start index = 300 — overlapping heavily. State sentence: `dp[i] = min total cost to run migrations i..n`. Recurrence: `dp[i] = min over j≥i of batchcost(i,j) + dp[j+1]`; base `dp[n] = 0`. States × transition = 300 × 300 = 9×10⁴ with O(1) batchcost via prefix sums. Implemented memoized (12 lines), validated against brute force at n=12 (caught an off-by-one in the prefix-sum bounds — the five-minute oracle check paying out), answer recovery via stored split points produced the actual batch plan. The plan shipped in the migration runbook; total risk cost 40% below the hand-made batching it replaced.

## Related skills
- `complexity-analysis` — the states×work bound and pseudo-polynomial warning.
- `graph-traversal-patterns` — DP on DAGs is traversal + values; cycles mean it's not a DP.
- `binary-search-variants` — "minimize the maximum" problems often prefer search-on-answer over DP.
- `property-based-testing` — the brute-force oracle validation in step 7.
