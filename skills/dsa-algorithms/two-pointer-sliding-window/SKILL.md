---
name: two-pointer-sliding-window
description: >
  Use when a problem over arrays/strings involves pairs, subarrays, or
  substrings and brute force is O(n²) — recognizing when two-pointer or
  sliding-window applies and instantiating the templates. Triggers:
  "longest substring with", "subarray sum", "pair that sums to", "container
  with most water", "minimum window", "remove duplicates in place".
---

# Two-Pointer / Sliding Window

## When to use this skill
- The problem asks about contiguous subarrays/substrings, pairs in sorted data, or in-place array rewriting — and the naive answer enumerates O(n²) candidates.
- Interview prep on this pattern family.
- NOT when the subarray constraint isn't monotonic (see step 5's applicability test) — those need prefix sums + hash maps or DP instead.

## Prerequisites
- `complexity-analysis` instincts — the pattern's payoff is O(n²)→O(n), argued via "each pointer moves forward ≤ n times total."

## Workflow

1. **Classify the problem into one of the three shapes:**
   - **Opposite ends, converging** (two-pointer proper): pairs in *sorted* data (two-sum-sorted, 3-sum), symmetric checks (palindrome), max-area (container problem). Pointers start at both ends, move inward by a greedy rule.
   - **Same direction, reader/writer:** in-place compaction (remove duplicates/zeros, keep-matching) — write pointer trails read pointer.
   - **Window, expanding/shrinking:** "longest/shortest/count of contiguous runs satisfying a condition" — right pointer expands, left pointer shrinks to restore validity.

2. **Converging template — the argument matters more than the code.** For sorted two-sum: `sum too small → left++` is justified because with `right` fixed at its current value, no *smaller* left partner can work either. Every converging solution needs this one-line justification of "why is discarding this pointer's position safe?" — if you can't state it, the pattern may not apply (see the container problem, where the shorter wall moves because keeping it caps all future areas).

3. **Window template — memorize one, instantiate many:**
   ```python
   best = 0; left = 0; state = {}          # state: whatever validity needs
   for right, x in enumerate(arr):
       add(state, x)                        # window is arr[left..right]
       while invalid(state):                # shrink until valid again
           remove(state, arr[left]); left += 1
       best = max(best, right - left + 1)
   ```
   The two design decisions per problem: what is `state` (counts dict, distinct count, sum) and what is `invalid`. Longest-substring-without-repeats: state = last-seen index or counts, invalid = a count > 1. Complexity: left and right each advance ≤ n → O(n), *provided* add/remove/invalid are O(1) — a state needing a re-scan per step silently restores O(n²).

4. **The min-window variant flips the loop:** shrink while VALID (recording the best at each valid shrink), for "shortest subarray/substring containing X." The tell for which variant: "longest ... while condition holds" = shrink-on-invalid; "shortest ... that achieves condition" = shrink-while-valid.

5. **Run the applicability test before committing — the monotonicity check.** Sliding window requires: *extending* a window moves the condition in one known direction, *shrinking* moves it back (adding elements only increases sum — works for positive numbers; adding chars only adds distinct chars — works). The classic failure: "subarray sum = k" **with negative numbers** — extending might raise or lower the sum, the shrink rule has no valid greedy direction → the right tool is prefix-sums + hash map (O(n), different pattern). Negative numbers / non-monotonic predicates are the #1 way this pattern gets misapplied confidently.

6. **For counting problems ("number of subarrays with..."), use the at-most trick:** count(exactly k) = count(at most k) − count(at most k−1), where "at most" fits the standard window. Direct counting inside a window double-counts; this decomposition is the standard escape.

7. **Verify with the boundary set:** empty input, single element, all-identical elements, the answer spanning the entire array, no valid answer (define the return!), and — for the window — the case where the window never shrinks and where it shrinks to empty. Off-by-one on `right - left + 1` vs `right - left` is the pattern's signature bug; pin the invariant ("window is inclusive `[left, right]`") in a comment and derive the length from it.

## Common pitfalls
- Applying the window to non-monotonic conditions (negatives in sums, "exactly k" directly) — the code runs, produces plausible wrong answers, and passes the happy-path tests. Step 5's check is mandatory.
- Converging pointers on *unsorted* data — two-sum's O(n) two-pointer requires the sort (O(n log n)) or a hash set (O(n)); pattern-matching the shape without the precondition.
- State maintenance asymmetry: adding is handled, removal forgets to delete zero-count keys → `invalid()` reads stale state. The add/remove pair must be true inverses; test them as such.
- The while-shrink written as an if — one shrink step per expansion isn't enough when one new element invalidates by a lot; the window drifts invalid and the answer is garbage.
- Reader/writer compaction that reads *after* writing over it — the two pointers' order guarantee (`write ≤ read`) is the safety proof; violating it with lookahead breaks in-place-ness.
- Claiming O(n) while `invalid()` scans the state dict per step (step 3's proviso).

## Example
Task (real code, not interview): rate-limiter audit — "longest span of requests from one client where >90% were errors," 10M-row log. Naive: all O(n²) spans, ~10¹³ ops, dead. Shape: longest-contiguous-with-condition → window. Monotonicity check: extending adds an error or a success — the *ratio* isn't monotonic! Direct window fails step 5. Reformulation: map success→−9, error→+1; ">90% errors" ⇔ window sum > 0 — with negatives, still not window-able → prefix-sum + monotonic-stack-style earliest-smaller-prefix lookup, O(n). Shipped in seconds of runtime. The lesson the example carries: the reformulation-then-recheck loop (steps 1→5→reformulate) IS the skill; pattern-forcing the window on the raw ratio would have produced confident nonsense.

## Related skills
- `complexity-analysis` — the amortized "each pointer moves n times" argument.
- `binary-search-variants` — the other "shrink the search space by invariant" family.
- `heap-and-priority-patterns` — windows needing max/min *of the window* (sliding-window-maximum: monotonic deque territory).
