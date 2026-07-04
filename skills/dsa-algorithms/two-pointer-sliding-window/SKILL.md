---
name: two-pointer-sliding-window
description: >
  Use when a problem over arrays/strings involves pairs, subarrays, or
  substrings and brute force is O(n²) — recognizing when two-pointer or
  sliding-window applies and instantiating the templates. Triggers:
  "longest substring with", "subarray sum", "pair that sums to",
  "container with most water", "minimum window", "remove duplicates in
  place". NOT when the subarray condition isn't monotonic (negatives in
  sums, "exactly k" directly) — those need prefix sums + hash maps or DP.
---

# Two-Pointer / Sliding Window

## Overview

The pattern converts O(n²) candidate enumeration into O(n) by arguing "each pointer only moves forward" — and it's valid only when the condition is monotonic under extend/shrink. The applicability test matters more than the template: misapplied windows produce confident wrong answers that pass happy-path tests.

## When to Use

- Contiguous subarray/substring questions, pairs in sorted data, in-place array rewriting — where the naive answer enumerates O(n²) candidates.
- Interview prep on this pattern family.

**When NOT to use:**
- Non-monotonic conditions: subarray-sum-equals-k WITH negative numbers, ratios, "exactly k" asked directly — prefix sums + hash maps or `dynamic-programming-derivation` instead. The step-5 test decides.

## Prerequisites

- `complexity-analysis` instincts — the payoff argument is "each pointer moves forward ≤ n times total."

## The Workflow

1. **Classify the problem into one of the three shapes:**
   - **Opposite ends, converging:** pairs in *sorted* data (two-sum-sorted, 3-sum), symmetric checks (palindrome), max-area. Pointers start at both ends, move inward by a greedy rule.
   - **Same direction, reader/writer:** in-place compaction (remove duplicates/zeros) — write pointer trails read pointer; the safety proof is `write ≤ read`, never violated by lookahead.
   - **Window, expanding/shrinking:** "longest/shortest/count of contiguous runs satisfying a condition" — right expands, left shrinks to restore validity.

2. **Converging template — the argument matters more than the code.** For sorted two-sum: `sum too small → left++` is justified because with `right` fixed, no *smaller* left partner can work either. Every converging solution needs this one-line "why is discarding safe?" justification — if you can't state it, the pattern may not apply. (Container problem: the shorter wall moves because keeping it caps all future areas.)

3. **Window template — memorize one, instantiate many:**
   ```python
   best = 0; left = 0; state = {}          # state: whatever validity needs
   for right, x in enumerate(arr):
       add(state, x)                        # window is arr[left..right] inclusive
       while invalid(state):                # WHILE, not if — one new element
           remove(state, arr[left]); left += 1   # can invalidate by a lot
       best = max(best, right - left + 1)
   ```
   Two design decisions per problem: what is `state`, what is `invalid`. Complexity is O(n) *provided* add/remove/invalid are O(1) — a state needing a re-scan per step silently restores O(n²). And `add`/`remove` must be true inverses (delete zero-count keys, or `invalid` reads stale state).

4. **The min-window variant flips the loop:** shrink while VALID (recording best at each valid shrink), for "shortest subarray containing X." The tell: "longest ... while condition holds" = shrink-on-invalid; "shortest ... achieving condition" = shrink-while-valid.

5. **Run the applicability test before committing — the monotonicity check:**

   ```
   Does EXTENDING the window move the condition in one
   known direction, and SHRINKING move it back?
        |                        |
       yes                       no
        |                        |
        v                        v
   window applies         STOP — reformulate:
   (sums of positives,    prefix-sum + hash map
    distinct counts)      (negatives), at-most trick
                          ("exactly k"), or DP
   ```

   The classic failure: "subarray sum = k" with negatives — extending might raise or lower the sum; no valid greedy shrink direction exists. Negative numbers and non-monotonic predicates are the #1 confident misapplication.

6. **For counting problems ("number of subarrays with..."), use the at-most trick:** count(exactly k) = count(at most k) − count(at most k−1), where "at most" fits the standard window. Direct counting inside a window double-counts.

7. **Verify with the boundary set:** empty input, single element, all-identical, answer spanning the whole array, no valid answer (define the return!), window never shrinking, window shrinking to empty. Off-by-one on `right - left + 1` is the signature bug; pin the invariant ("window is inclusive [left, right]") in a comment and derive the length from it.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "It's a subarray problem — sliding window applies" | Shape-matching skips the monotonicity test, and negatives/ratios break the shrink rule silently. The window on a non-monotonic condition RUNS — it just returns plausible wrong answers. |
| "My tests pass, the window must be valid" | Happy-path tests rarely include the input where the greedy shrink was wrong. Validity comes from the step-5 argument, not from green tests on friendly data. |
| "I'll use an if instead of while for the shrink — faster" | One new element can invalidate by a lot; single-step shrinking lets the window drift invalid and every subsequent answer is garbage. The while IS the invariant restoration. |
| "Two pointers work on this array once I sort it" | Sorting costs O(n log n) and DESTROYS index information — if the answer needs original positions, the sort just broke the problem. Check what the output requires first. |
| "The state dict re-scan is fine, it's small" | Per-step re-scans restore O(n²) while the code still 'looks O(n)'. The complexity claim rides on O(1) state ops — the proviso is load-bearing. |
| "Exactly-k is just the window with equality" | Direct exactly-k double-counts and has no monotone shrink. The at-most decomposition (step 6) is the standard escape, not an optimization. |

## Red Flags

- A window solution on a problem whose inputs include negatives, with no monotonicity argument anywhere.
- No stated justification for the converging pointers' discard rule.
- `if invalid` where `while invalid` belongs.
- `remove` that decrements counts but never deletes zero-count keys.
- Length computed inconsistently (`right - left` here, `+1` there).
- The boundary battery (empty/single/whole-array/no-answer) absent from tests.

## Verification

- [ ] Shape classified (converging / reader-writer / window) — named in the solution notes.
- [ ] Monotonicity or discard-safety argument written in one line — comment or PR.
- [ ] State add/remove verified as inverses — test with add-then-remove returning to baseline.
- [ ] Complexity claim checked: all state ops O(1) — noted.
- [ ] Boundary battery passing: empty, single, all-same, full-span, no-answer — test names listed.
- [ ] For counting problems: at-most decomposition used, not direct exactly-k.

## Example

Real task: rate-limiter audit — "longest span of requests from one client where >90% were errors," 10M-row log. Naive: all O(n²) spans, ~10¹³ ops, dead. Shape: longest-contiguous-with-condition → window. Monotonicity check: extending adds an error or a success — the *ratio* isn't monotonic! Direct window fails step 5. Reformulation: map success→−9, error→+1; ">90% errors" ⇔ window sum > 0 — with negatives, still not window-able → prefix-sum + earliest-smaller-prefix lookup, O(n). Shipped in seconds of runtime. The lesson: the reformulate-then-recheck loop IS the skill; pattern-forcing the window on the raw ratio would have produced confident nonsense.

## Related skills

- `complexity-analysis` — the amortized "each pointer moves n times" argument.
- `binary-search-variants` — the other "shrink the search space by invariant" family.
- `heap-and-priority-patterns` — windows needing max/min OF the window (monotonic deque territory).
- `dynamic-programming-derivation` — where non-monotonic subarray problems often land.
