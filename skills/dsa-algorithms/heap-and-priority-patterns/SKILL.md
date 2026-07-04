---
name: heap-and-priority-patterns
description: >
  Use when a problem repeatedly needs the current min/max — top-k
  selection, merging k sorted streams, scheduling by priority, running
  medians — with heap templates and the size-k trick. Triggers: "top k",
  "k largest", "merge k sorted", "task scheduling by priority", "running
  median", "kth smallest", "process in priority order". NOT for one-shot
  min/max (a scan), full sorting (just sort), or lookup-heavy mixes (see
  data-structure-selection's table).
---

# Heap & Priority Patterns

## Overview

Heaps earn their keep on *interleaved* streams and *k ≪ n* selections — everywhere else, a sort is simpler and cache-friendlier. The two library gotchas (min-only heaps, incomparable payloads on tied keys) and the size-k inversion are where most heap code goes wrong before any algorithm does.

## When to Use

- "Repeatedly extract the best" in the requirements: top-k, k-way merge, priority scheduling, sliding medians.

**When NOT to use:**
- One-shot min/max — a scan; fully-batched data — a sort; lookup-heavy mixes — `data-structure-selection`'s table (heaps don't do arbitrary lookup).

## Prerequisites

- The op-mix habit from `data-structure-selection`: heaps pay off when insertions interleave with best-extractions.

## The Workflow

1. **Know the two library gotchas before any pattern:** most stdlib heaps are min-heaps only (`heapq`) — max-heap = negate keys; and heap entries with equal keys need a tiebreaker when payloads aren't comparable — the standard entry is `(key, seq, payload)` with an incrementing counter:
   ```python
   heappush(h, (priority, next(counter), task))   # counter breaks ties; task never compared
   ```
   The seq-counter is not hygiene — it's a latent prod crash on the first tied priority with dict payloads.

2. **Top-k: the inverted-heap-of-size-k trick — the pattern's crown jewel.** K largest from n (or a stream): keep a MIN-heap of size k; each new item compares against the heap's minimum (the weakest of the current top-k):
   ```python
   h = nums[:k]; heapq.heapify(h)
   for x in nums[k:]:
       if x > h[0]: heapq.heapreplace(h, x)
   ```
   O(n log k) time, O(k) space — works on streams too big to hold. The inversion (MIN-heap for k-LARGEST) is the part everyone gets backwards once: the heap holds candidates, so its root is the eviction boundary. Shortcuts: `heapq.nlargest`; quickselect for one-shot arrays (O(n) average) when streaming isn't needed.

3. **K-way merge: heap of (head, source) pairs.** Merging k sorted lists/streams/files: seed with each source's first element, pop the global min, push that source's next:
   ```python
   h = [(lst[0], i, 0) for i, lst in enumerate(lists) if lst]
   heapq.heapify(h)
   while h:
       val, src, idx = heappop(h); out.append(val)
       if idx + 1 < len(lists[src]): heappush(h, (lists[src][idx+1], src, idx+1))
   ```
   O(N log k). This IS external merge sort's core, LSM compaction, and "merge k event streams by timestamp" — the most production-relevant heap pattern (`heapq.merge` packages it).

4. **Scheduling: heap keyed on next-eligible-time or priority.** Push `(ready_time, seq, task)`, pop when `now >= ready_time`; two-heap variants (waiting-by-time + eligible-by-priority) handle "highest priority among the currently-eligible" — the shape of real schedulers and interval problems (meeting-rooms: min-heap of end-times; heap size = rooms in use).

5. **Running median: two balanced heaps.** Max-heap of the lower half + min-heap of the upper, rebalanced to size difference ≤ 1; median = a root. Invariant: every lower-heap element ≤ every upper-heap element — push to one side, rebalance by moving a root. Generalizes to running percentiles (size ratio p : 1−p).

6. **Arbitrary deletion/update: lazy tombstones, not surgery.** Heaps can't delete by value efficiently; the production answer is the tombstone set — mark deleted, discard on pop. For decrease-key patterns (Dijkstra): push the updated entry, tombstone-skip stale pops — simpler than indexed heaps and usually as fast. Keep the tombstone set bounded (drain on pops) or the "deleted" majority bloats the heap. Never mutate an element's key in place — silently breaks the invariant; every subsequent pop is untrustworthy.

7. **State the complexity honestly and check the alternative:** heapify is O(n) (not n log n — building beats n pushes); n pushes+pops = O(n log n) ≈ a sort — so if everything is pushed before anything is popped, sort instead. Say which case applies or use the sort.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "Max-heap of size k for the k largest — obviously" | Backwards: the max-heap of size k keeps the k SMALLEST seen (it evicts from the top). The min-heap's root is the eviction boundary for k-largest — the inversion bites everyone exactly once; make it in review, not prod. |
| "(priority, task_dict) entries work fine in my tests" | Until the first tied priority forces Python to compare the dicts — TypeError in production. The seq-counter (step 1) costs one line; its absence costs an incident. |
| "The heap makes finding the max efficient" | Finding the max ONCE is an O(n) scan; heapify costs O(n) too and adds machinery. Heaps amortize repeated extraction; single extraction is scan territory. |
| "All the data arrives, then we query — heap it anyway" | n pushes + n pops = O(n log n) = a sort, with worse constants and cache behavior. Batch-then-sort wins whenever nothing interleaves (step 7's check). |
| "I'll just update the entry's priority in place" | In-place key mutation breaks the heap invariant silently — every later pop is a lie. Tombstone + re-push (step 6) is the pattern precisely because surgery isn't. |
| "Tombstones will get cleaned up eventually" | Under churn, 'eventually' means a heap that's 90% garbage paying log(garbage) per op. Bound the set (drain on pop, version counters) or the fix becomes the bloat. |

## Red Flags

- `(priority, payload)` tuples with non-comparable payloads and no tiebreaker.
- A size-k heap whose polarity nobody can justify in one sentence.
- Heap built, fully drained, nothing interleaved — a sort in costume.
- Key mutation on elements sitting in a live heap.
- An unbounded tombstone/cancelled set beside a long-lived heap.
- Sliding-window max via heap with no note that the monotonic deque upgrade exists.

## Verification

- [ ] Entry shape includes the tiebreaker wherever payloads aren't comparable — code inspected.
- [ ] Top-k polarity justified: one sentence naming the root as eviction boundary.
- [ ] Interleaving confirmed (or the sort alternative consciously rejected) — noted.
- [ ] Deletion/update via tombstones with a boundedness mechanism — described.
- [ ] Complexity stated with heapify-is-O(n) correctly used.
- [ ] Load/perf numbers at realistic scale for hot-path heaps — attached.

## Example

Real task: alert-storm dampener — from ~2M events/hour, maintain "the 50 highest-severity unresolved alerts," where alerts also *resolve* (removal) and *escalate* (key change). Design: full min-heap keyed on `(-severity, seq)` with a tombstone set for resolved/stale entries, drained on pop; dashboard snapshots pop-and-peek the top 50 every 5s, re-pushing live entries. Escalations: push the new entry, tombstone the old `(alert_id, version)`. Tombstone bound: version counter per alert makes staleness checkable in O(1), set pruned on read. Load test at 10× storm volume: snapshot p99 = 4ms. The first draft had used `(priority, alert_dict)` entries — crashed on tied severities within the hour, exactly per step 1's warning; the seq counter fixed it in one line.

## Related skills

- `data-structure-selection` — when the answer is NOT a heap.
- `two-pointer-sliding-window` — window problems where a monotonic deque supersedes the heap.
- `graph-traversal-patterns` — Dijkstra as scheduling-by-distance with lazy deletion.
- `complexity-analysis` — the heapify-is-O(n) class of bound facts.
