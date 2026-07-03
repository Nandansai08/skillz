---
name: complexity-analysis
description: >
  Use when deriving the time/space complexity of code — nested structures,
  amortized costs, hidden costs inside library calls, and when big-O stops
  mattering. Triggers: "what's the complexity of this", "is this O(n log n)",
  "why is this slow at scale", "amortized analysis", "will this scale to
  a million items".
---

# Complexity Analysis

## When to use this skill
- Judging whether code survives its input growing 100×.
- Interview prep, or settling a review debate about a nested loop.
- NOT for measuring actual runtime — that's profiling; big-O predicts the *shape* of growth, the profiler measures the reality.

## Prerequisites
- Know what n actually is for this code — items? characters? users × items? Half of wrong analyses start by miscounting the input.

## Workflow

1. **Name every input size first.** n rows, m columns, k distinct keys, L average string length. Complexity in one variable when two matter is the top-tier error: "O(n) over the list" hides that each step compares strings of length L → O(nL). Write the variables down before counting anything.

2. **Count the dominant operation through the structure:** sequential blocks add (keep the max), nested loops multiply *only if the inner bound is independent* — the inner loop running `for j in range(i)` gives n(n−1)/2 → O(n²), but an inner loop over *each item's children* across the whole outer loop touches each child once → O(total children), not O(n × max_children). "Total work across all iterations" beats "per-iteration × iterations" whenever the inner bound varies.

3. **Expand the hidden costs — the calls that look O(1):**
   - `list.insert(0, x)` / `arr.shift()` — O(n) each; in a loop, O(n²).
   - `x in list` — O(n); `x in set/dict` — O(1) avg. The single most common accidental quadratic: membership test in a loop.
   - String concatenation in a loop (`s += piece`) — O(n²) total in most languages; join/builder is O(n).
   - `sorted()`, `.sort()` — O(n log n), obviously, but also *inside* someone's helper you called per-row.
   - Slicing/copying (`arr[1:]`, spread operators) — O(n) per call; recursive functions slicing per level are how "O(n) recursion" becomes O(n²).
   - Regex with catastrophic backtracking — exponential on adversarial input (also a DoS — `secure-code-review` step 6's ReDoS).

4. **Recursion: draw the recurrence, then solve it by pattern.** T(n) = aT(n/b) + f(n) → master-theorem cases: halve-and-do-constant-work = O(log n) (binary search); two-halves-plus-linear-merge = O(n log n) (mergesort); double-branch-minus-one (T(n−1) twice, naive Fibonacci) = O(2ⁿ) — memoization collapses it to the number of *distinct states* × work per state, which is the analysis that matters for `dynamic-programming-derivation`.

5. **Amortized: charge expensive rare operations against cheap frequent ones.** Dynamic array doubling: n appends cost O(n) *total* (each doubling's copy is paid for by the appends that filled it) → O(1) amortized per append, even though one append occasionally costs O(n). Same reasoning: hash-map resize, two-stack queues, union-find with path compression (effectively constant). Use amortized bounds for throughput questions; use worst-case-per-operation when a single slow operation matters (real-time systems, p99 latency — `alerting-design` world cares about the spike amortization hides).

6. **Space: count what's alive simultaneously, including the call stack.** Recursion depth is space (n-deep recursion on a linked list = O(n) stack, and a real stack-overflow at ~10⁴–10⁵ frames in most runtimes); generators/iterators vs materialized lists is often THE space decision; "in-place" claims deserve an audit of temporaries.

7. **Sanity-check against the budget: n and the constant both matter.** The practical table — at ~10⁸ simple ops/sec: n=10³ tolerates O(n²); n=10⁵ needs O(n log n); n=10⁷ needs O(n) with a decent constant; n=10⁹ needs O(log n)/streaming. And below ~10³, constants beat asymptotics: linear scan outruns hash lookup on 30 items (cache locality), insertion sort beats quicksort under ~16 elements (why real sorts hybridize). Big-O picks the right neighborhood; the profiler picks the house.

## Common pitfalls
- Two nested loops declared O(n²) when the inner loop's *total* work across the outer is O(n) (step 2's dependence check) — sliding-window and adjacency-list traversals are the canonical false quadratics.
- The `in list` inside a loop shipping to production, fine at n=100 in dev, 40 seconds at n=10⁵ (`environment-parity` step 4's data-shape gap, algorithmic edition).
- Averaging away the worst case where it bites: hash-map O(1) "average" under adversarial keys (hash-flooding DoS), quicksort O(n log n) "average" on sorted input with naive pivots.
- Analyzing n while the real growth is in k (distinct keys) or L (string length) — the one-variable trap from step 1.
- Optimizing the O(n) parser while the O(n²) deduplication next to it owns 98% of runtime at scale — complexity analysis ranks the *suspects*; the profiler convicts.
- Trusting "the library handles it": `pandas.merge` is fine; calling it inside `df.apply` per-row is the quadratic you wrote, wearing the library's badge.

## Example
Review question: "will this dedupe survive 1M records?" Code: for each record, `if r.email not in seen_list: seen_list.append(r.email)`, plus a per-record `sorted(r.tags)`. Analysis: variables n=1M records, t=avg 5 tags. The `in list` is O(n) per check → O(n²) = 10¹² ops — weeks, not seconds. The sort is O(t log t) per record → O(n · t log t) ≈ 10⁷ — irrelevant. Fix: `seen = set()` → membership O(1), total O(n·L) for L-char emails ≈ 10⁷–10⁸ — seconds. The reviewer's instinct had been to worry about the sort; the analysis pointed one line left. Shipped with a comment: `# set, not list: O(n²) at production scale otherwise`.

## Related skills
- `data-structure-selection` — choosing the structure that fixes the bound.
- `dynamic-programming-derivation` — step 4's state-counting, systematized.
- `sql-query-optimization` — the same discipline where the loop is a join.
