---
name: binary-search-variants
description: >
  Use when halving a search space — leftmost/rightmost boundaries in sorted
  data, search on the answer space, and off-by-one-proof templates.
  Triggers: "binary search bug", "first/last occurrence", "minimize the
  maximum", "smallest X such that", "search in rotated array",
  "infinite loop in my binary search".
---

# Binary Search Variants

## When to use this skill
- Sorted (or monotonically-checkable) data needs a boundary, not just a hit.
- An optimization problem asks "smallest/largest value satisfying a feasibility condition."
- A hand-rolled binary search is off-by-one or looping forever (the genre's two diseases).
- NOT for unsorted membership — that's a hash set (`data-structure-selection`).

## Prerequisites
- The monotonic predicate identified: some `check(x)` that flips from False to True exactly once across the space. No monotone flip = no binary search, however sorted things look.

## Workflow

1. **Reframe every variant as boundary-finding on a predicate.** Not "find the target" but "find the first index where `arr[i] >= target`" (lower bound) or "first where `arr[i] > target`" (upper bound). Exact-match search, first/last occurrence, insertion point, floor/ceiling — all are one of these two boundaries plus a final equality check. One mental model, zero special cases.

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
   Why this template is bug-proof: half-open `[lo, hi)`, `mid` computed to floor (so `mid < hi` always, and `lo = mid + 1` guarantees progress → no infinite loop), and the two updates map exactly to "keep candidate" vs "exclude." Every off-by-one bug in the genre is a deviation from one of these three choices. `lo + (hi-lo)//2` also sidesteps overflow in fixed-width languages.

3. **Check your language's stdlib first:** `bisect_left`/`bisect_right` (Python), `lower_bound`/`upper_bound` (C++), `Collections.binarySearch`, `sort.Search` (Go — which IS the first_true template). Hand-rolling what `bisect` does is a bug opportunity with no upside; hand-roll only when the predicate isn't array indexing.

4. **Search on the answer space — the variant that solves optimization problems.** Pattern: "minimize X such that feasible(X)" where feasibility is monotone (if X works, X+1 works). Examples: minimum ship capacity to deliver in D days, minimum eating speed, smallest divisor, max-min allocation. The space is the *answer's* range, the predicate is a feasibility simulation:
   ```python
   # min capacity: check(c) = "can ship in D days at capacity c" (greedy sim, O(n))
   answer = first_true(max(weights), sum(weights) + 1, check)
   ```
   Design checklist: prove `check` monotone (one sentence), set `lo` = smallest possibly-feasible, `hi` = one past a certainly-feasible value, cost = O(check) × log(range).

5. **Real-valued spaces: iterate a fixed count, don't chase epsilon.** `for _ in range(100): mid = (lo+hi)/2 ...` — 100 halvings cover any double's precision; epsilon-based `while hi-lo > 1e-9` loops forever when the interval hits float granularity below epsilon.

6. **Rotated/bitonic and friends: identify which half is clean.** Rotated sorted array: at each mid, one side is properly sorted (check its endpoints); decide membership against the clean side and recurse into the right half. Duplicates poison the guarantee (`[1,1,1,0,1]` — can't tell which side is clean) → worst case degrades to O(n); say so rather than pretending.

7. **Test the boundary battery — six inputs kill 95% of implementations:** empty range, single element, all elements satisfy, none satisfy, target at index 0, target at last index. Plus, for answer-space searches: `check(lo)` already true, and `hi` genuinely unreachable. If writing property tests, the oracle is linear scan (`property-based-testing`'s pattern — binary search vs `next(i for i ...)` on random arrays).

## Common pitfalls
- Three-way branching (`if ==: return; elif <: ...`) plus inclusive `hi = len-1` plus `mid ± 1` decisions made ad hoc — the classic hand-roll where each of three choices can be off by one, multiplied. The template collapses the decision space to zero.
- Infinite loop from `lo = mid` with floor division (`mid` recomputes to `lo` forever when `hi = lo+1`). The template's `lo = mid + 1` / ceil-division-when-searching-rightmost duality; deviate knowingly or not at all.
- Predicate not actually monotone: "days to complete at speed k" is monotone; "profit at price p" typically isn't (rises then falls — that's ternary search or calculus, not binary). One counterexample check before coding saves the debugging hour.
- Returning `lo` without handling "no True exists" — `first_true` returns the original `hi`; callers must check. Silent out-of-range indexing follows otherwise.
- Answer-space bounds guessed too tight: `hi = sum(weights)` when the answer could BE `sum(weights)` and the half-open template needs `hi` one past it. Boundary semantics apply to the bounds themselves.
- Binary searching a linked structure (no O(1) random access) — log n *probes* × O(n) access = worse than the scan.

## Example
Real task: autoscaler tuning — "smallest replica count where p99 latency ≤ 200ms under the recorded peak trace," where each `check(replicas)` is a 3-minute load-test replay (`capacity-planning` step 3's knee, found by search). Space: 2..128 replicas. Monotonicity argued: more replicas never raises latency for this stateless tier (shared DB pool capped separately — the caveat written down, because a pool bottleneck WOULD break monotonicity and the search). `first_true(2, 129, check)` = 7 probes instead of 127 sequential runs: 21 minutes instead of 6 hours. Answer: 44 replicas; the boundary battery's "check(lo) already true" case caught a harness bug where 2 replicas "passed" because the load generator itself was the bottleneck — fixed before the search, not after the rollout.

## Related skills
- `two-pointer-sliding-window` — the other invariant-driven index dance.
- `dynamic-programming-derivation` — "minimize the max" problems where search-on-answer replaces a harder DP.
- `debugging-by-bisection` — the same halving discipline applied to commits and inputs.
- `capacity-planning` — the example's home domain.
