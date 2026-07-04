---
name: complexity-analysis
description: >
  Use when deriving the time/space complexity of code — nested structures,
  amortized costs, hidden costs inside library calls, and when big-O
  stops mattering. Triggers: "what's the complexity of this", "is this
  O(n log n)", "why is this slow at scale", "amortized analysis", "will
  this scale to a million items". NOT for measuring actual runtime —
  that's profiling; big-O predicts the shape of growth, the profiler
  measures the reality.
---

# Complexity Analysis

## Overview

Complexity analysis predicts whether code survives its input growing 100× — and its most common failures are miscounting which variable grows, and missing the O(n) hiding inside an innocent-looking call. Big-O ranks the suspects; the profiler convicts.

## When to Use

- Judging whether code survives its input growing 100×.
- Interview prep, or settling a review debate about a nested loop.

**When NOT to use:**
- Measuring actual runtime — profiling; the two answer different questions.

## Prerequisites

- Know what n actually is for this code — items? characters? users × items? Half of wrong analyses start by miscounting the input.

## The Workflow

1. **Name every input size first.** n rows, m columns, k distinct keys, L average string length. Complexity in one variable when two matter is the top-tier error: "O(n) over the list" hides that each step compares strings of length L → O(nL). Write the variables down before counting anything.

2. **Count the dominant operation through the structure:** sequential blocks add (keep the max), nested loops multiply *only if the inner bound is independent* — an inner loop over *each item's children* across the whole outer loop touches each child once → O(total children), not O(n × max_children). "Total work across all iterations" beats "per-iteration × iterations" whenever the inner bound varies — sliding windows and adjacency traversals are the canonical false quadratics.

3. **Expand the hidden costs — the calls that look O(1):**
   - `list.insert(0, x)` / `arr.shift()` — O(n) each; in a loop, O(n²).
   - `x in list` — O(n); `x in set/dict` — O(1) avg. The single most common accidental quadratic: membership test in a loop.
   - String concatenation in a loop (`s += piece`) — O(n²) total in most languages.
   - `sorted()` — O(n log n), including *inside* someone's helper you call per-row.
   - Slicing/copying (`arr[1:]`, spreads) — O(n) per call; recursive functions slicing per level turn O(n) recursion into O(n²).
   - Regex with catastrophic backtracking — exponential on adversarial input (also a DoS — ReDoS).

4. **Recursion: draw the recurrence, then solve by pattern.** T(n) = aT(n/b) + f(n) → master-theorem cases: halve-and-constant-work = O(log n); two-halves-plus-linear-merge = O(n log n); double-branch (naive Fibonacci) = O(2ⁿ) — memoization collapses it to distinct states × work per state, the analysis `dynamic-programming-derivation` lives by.

5. **Amortized: charge expensive rare operations against cheap frequent ones.** Dynamic-array doubling: n appends cost O(n) *total* → O(1) amortized, though one append occasionally costs O(n). Same reasoning: hash resize, two-stack queues, union-find with path compression. Use amortized for throughput; use worst-case-per-operation when a single slow op matters (real-time, p99 latency — the spike amortization hides is exactly what tail-latency work cares about).

6. **Space: count what's alive simultaneously, including the call stack.** Recursion depth is space (n-deep recursion = O(n) stack, and real stack-overflow at ~10⁴–10⁵ frames); generators vs materialized lists is often THE space decision; "in-place" claims deserve an audit of temporaries.

7. **Sanity-check against the budget: n and the constant both matter.** At ~10⁸ simple ops/sec: n=10³ tolerates O(n²); n=10⁵ needs O(n log n); n=10⁷ needs O(n); n=10⁹ needs O(log n)/streaming. And below ~10³, constants beat asymptotics: linear scan outruns hash lookup on 30 items; insertion sort beats quicksort under ~16 (why real sorts hybridize). Big-O picks the neighborhood; the profiler picks the house.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "Two nested loops — it's O(n²), flag it" | If the inner loop's total work across the outer is O(n) (children, windows), it's linear. Dependence-checking the inner bound is the analysis; loop-counting is pattern-matching. |
| "It's fast in dev, ship it" | n=100 in dev forgives the `in list` that takes 40 seconds at n=10⁵. Complexity is precisely the property dev-scale testing cannot see. |
| "The library call handles it efficiently" | pandas.merge is fine; calling it inside df.apply per-row is the quadratic YOU wrote wearing the library's badge. Hidden costs (step 3) live inside exactly the calls that look innocent. |
| "Average case is O(1), so we're fine" | Hash maps under adversarial keys and quicksort on sorted input both say hello. Average-case reasoning needs a 'who controls the input?' check before it's load-bearing. |
| "I'll optimize the algorithm — it's the complex-looking part" | The O(n) parser next to an O(n²) dedupe owns 2% of runtime at scale. Analysis ranks suspects; optimizing the interesting-looking one is how weeks get spent on 2%. |
| "Asymptotics say the tree beats the array here" | At n=30, the array's cache locality wins by multiples. Below ~10³, constants govern — the budget table (step 7) exists to stop asymptotic overreach. |

## Red Flags

- An analysis in one variable for code with two growth dimensions.
- `in list`, `insert(0,…)`, or loop-concatenation shipped inside hot loops.
- "O(1) average" claimed where input is attacker-influenced.
- Recursion depth unconsidered for deep-input data (the stack IS memory).
- Optimization effort allocated with no complexity ranking of the candidates.
- A big-O argument used to reject a solution at n=50.

## Verification

- [ ] Input variables named (n, m, k, L...) before any counting — visible in the analysis.
- [ ] Inner-loop dependence checked for every nested structure — total-work argument stated where bounds vary.
- [ ] Hidden-cost sweep done on library/builtin calls in hot paths — flagged ones listed.
- [ ] The verdict tied to the actual budget (n at production scale vs the step-7 table) — one sentence.
- [ ] For fixes: before/after measurement at realistic scale confirms the predicted shape — numbers attached.

## Example

Review question: "will this dedupe survive 1M records?" Code: for each record, `if r.email not in seen_list: seen_list.append(r.email)`, plus a per-record `sorted(r.tags)`. Analysis: variables n=1M records, t=avg 5 tags. The `in list` is O(n) per check → O(n²) = 10¹² ops — weeks, not seconds. The sort is O(t log t) per record → ≈ 10⁷ — irrelevant. Fix: `seen = set()` → membership O(1), total O(n·L) ≈ 10⁷–10⁸ — seconds. The reviewer's instinct had been to worry about the sort; the analysis pointed one line left. Shipped with a comment: `# set, not list: O(n²) at production scale otherwise`.

## Related skills

- `data-structure-selection` — choosing the structure that fixes the bound.
- `dynamic-programming-derivation` — step 4's state-counting, systematized.
- `sql-query-optimization` — the same discipline where the loop is a join.
