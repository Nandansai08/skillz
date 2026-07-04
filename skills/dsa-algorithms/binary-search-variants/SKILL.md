---
name: binary-search-variants
description: >
  Use when halving a search space — leftmost/rightmost boundaries in
  sorted data, search on the answer space, and off-by-one-proof
  templates. Triggers: "binary search bug", "first/last occurrence",
  "minimize the maximum", "smallest X such that", "search in rotated
  array", "infinite loop in my binary search". NOT for unsorted
  membership — that's a hash set (see data-structure-selection).
---

# Binary Search Variants

## Overview

Every binary search is boundary-finding on a monotonic predicate — exact match, first occurrence, and "smallest capacity that works" are all the same `first_true` template. The genre's two diseases (off-by-one, infinite loop) are both cured by never improvising the loop invariant.

## When to Use

- Sorted (or monotonically-checkable) data needing a boundary, not just a hit.
- Optimization asking "smallest/largest value satisfying a feasibility condition."
- A hand-rolled binary search is off-by-one or looping forever.

**When NOT to use:**
- Unsorted membership — hash set; and linked structures (no O(1) random access) — the scan wins.

## Prerequisites

- The monotonic predicate identified: some `check(x)` that flips False→True exactly once across the space. No monotone flip = no binary search, however sorted things look.

## The Workflow

1. **Reframe every variant as boundary-finding on a predicate.** Not "find the target" but "find the first index where `arr[i] >= target`" (lower bound) or "first where `arr[i] > target`" (upper bound). Exact match, first/last occurrence, insertion point, floor/ceiling — all are one of these two boundaries plus a final equality check. One mental model, zero special cases.

2. **Use the one loop-invariant template and never improvise:**
   ```python
   def first_true(lo, hi, check):
       # invariant: check is False for all < lo; True for all >= hi (if any)
       while lo < hi:
           mid = lo + (hi - lo) // 2
           if check(mid): hi = mid          # mid might be the answer: keep it
           else:          lo = mid + 1      # mid is ruled out: exclude it
       return lo                             # first True, or original hi if none
   ```
   Why bug-proof: half-open `[lo, hi)`, floor-mid (so `mid < hi` always, and `lo = mid + 1` guarantees progress → no infinite loop), and the two updates map exactly to keep-candidate vs exclude. Every off-by-one in the genre is a deviation from one of these three choices. Callers must handle the "no True exists" return (original `hi`).

3. **Check your language's stdlib first:** `bisect_left`/`bisect_right`, `lower_bound`/`upper_bound`, `sort.Search` (Go — which IS the template). Hand-roll only when the predicate isn't array indexing.

4. **Search on the answer space — the variant that solves optimization problems.** Pattern: "minimize X such that feasible(X)" where feasibility is monotone (if X works, X+1 works). Examples: minimum ship capacity, minimum eating speed, max-min allocation:
   ```python
   # min capacity: check(c) = "can ship in D days at capacity c" (greedy sim, O(n))
   answer = first_true(max(weights), sum(weights) + 1, check)
   ```
   Design checklist: prove `check` monotone in one sentence; `lo` = smallest possibly-feasible; `hi` = ONE PAST a certainly-feasible value (half-open semantics apply to the bounds themselves); cost = O(check) × log(range).

5. **Real-valued spaces: iterate a fixed count, don't chase epsilon.** `for _ in range(100): ...` — 100 halvings exceed any double's precision; `while hi-lo > 1e-9` loops forever when the interval hits float granularity below epsilon.

6. **Rotated/bitonic and friends: identify which half is clean.** Rotated sorted array: at each mid, one side is properly sorted (check its endpoints); decide membership against the clean side, recurse into the other. Duplicates poison the guarantee (`[1,1,1,0,1]` — can't tell which side is clean) → worst case degrades to O(n); say so rather than pretending.

7. **Test the boundary battery — six inputs kill 95% of implementations:** empty range, single element, all-satisfy, none-satisfy, target at index 0, target at last index. Plus for answer-space: `check(lo)` already true, and hi genuinely unreachable. Property-test oracle: linear scan (`property-based-testing`'s pattern — binary search vs `next(i for i ...)` on random arrays).

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "I'll write the three-way if/elif version — it's clearer" | Three-way branching + inclusive hi + ad-hoc mid±1 = three independently improvised choices, each off-by-one-able. The template collapses the decision space to zero; 'clearer' here means 'more places to be wrong.' |
| "lo = mid is fine, it converges" | With floor division, `mid` recomputes to `lo` forever when `hi = lo+1` — the classic infinite loop. `lo = mid + 1` (or ceil-mid for rightmost searches) is the progress guarantee, not a style choice. |
| "The data is sorted, so binary search applies" | Sorted ≠ monotone predicate for YOUR question: 'profit at price p' rises then falls on sorted prices. One counterexample check on the predicate beats an hour of debugging the 'right' algorithm on the wrong shape. |
| "bisect doesn't do exactly what I need — I'll hand-roll" | bisect_left/right + one equality check cover exact-match, first/last, insertion, floor/ceiling. The gap is almost always framing, not functionality (step 1). |
| "hi = sum(weights) — the answer can't exceed that" | The answer can BE that, and half-open needs hi one past it. Boundary semantics apply to the bounds; the tight-hi bug returns wrong answers only at the extreme, i.e., in production. |
| "Epsilon loop for the real-valued search — standard" | Below float granularity, hi-lo stops shrinking past epsilon and the loop never exits. Fixed-count iteration is shorter AND terminates by construction. |

## Red Flags

- A hand-rolled search with `lo = mid` and floor division anywhere.
- Mixed conventions: inclusive hi with half-open updates.
- Answer-space search with no written monotonicity argument.
- The no-True-exists return value unhandled by the caller.
- Real-valued search with an epsilon while-loop.
- No boundary battery in the tests; the search "verified" on one happy case.

## Verification

- [ ] The predicate stated and its monotone flip argued in one sentence — comment.
- [ ] Template (or stdlib) used; any deviation from half-open/floor-mid/±1 justified in writing.
- [ ] Bounds semantics checked: lo possibly-feasible, hi one-past certainly-feasible.
- [ ] The six-input boundary battery passing — test names listed.
- [ ] For answer-space: check() cost stated; total = O(check)·log(range) noted.
- [ ] Property oracle vs linear scan where the stakes justify — test linked.

## Example

Real task: autoscaler tuning — "smallest replica count where p99 latency ≤ 200ms under the recorded peak trace," where each `check(replicas)` is a 3-minute load-test replay (`capacity-planning`'s knee, found by search). Space: 2..128 replicas. Monotonicity argued: more replicas never raises latency for this stateless tier (shared DB pool capped separately — the caveat written down, because a pool bottleneck WOULD break monotonicity and the search). `first_true(2, 129, check)` = 7 probes instead of 127 sequential runs: 21 minutes instead of 6 hours. Answer: 44 replicas; the boundary battery's "check(lo) already true" case caught a harness bug where 2 replicas "passed" because the load generator itself was the bottleneck — fixed before the search, not after the rollout.

## Related skills

- `two-pointer-sliding-window` — the other invariant-driven index dance.
- `dynamic-programming-derivation` — "minimize the max" where search-on-answer replaces a harder DP.
- `debugging-by-bisection` — the same halving discipline on commits and inputs.
- `capacity-planning` — the example's home domain.
