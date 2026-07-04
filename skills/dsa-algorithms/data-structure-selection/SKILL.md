---
name: data-structure-selection
description: >
  Use when choosing the data structure for an access pattern — the
  decision table from operations-needed to structure, with the real-world
  costs the textbook table omits. Triggers: "what data structure should I
  use", "dict or list here", "need fast lookup and ordering", "priority
  queue or sorted list", "how should I store this". NOT for database/
  storage-layer choices (see sql-query-optimization for indexes — the
  same table at disk scale).
---

# Data Structure Selection

## Overview

Structure follows access pattern, not the data's noun: the same "list of users" wants an array, a dict, a heap, or a deque depending on the operations performed on it. The op-mix written down decides in a minute what intuition debates for an hour.

## When to Use

- Code needs a container and the choice isn't obviously "just a list."
- A profiler/review flagged a structure misuse (the O(n) membership check).

**When NOT to use:**
- Database indexes and storage engines — `sql-query-optimization`.

## Prerequisites

- The operation mix, written down: which operations, roughly how often, at what n. "Insert-heavy, occasional lookup, n≈10⁵" decides more than any benchmark.

## The Workflow

1. **List the operations the code actually performs** — not the data's "natural shape." Structure follows access, not nouns.

2. **Match against the core table** (average cases; language names vary):

   | Need | Structure | Cost | The catch |
   |---|---|---|---|
   | Index access, iteration, append | dynamic array (`list`, `Vec`) | O(1) | insert/delete at front/middle O(n) |
   | Membership, lookup by key | hash map/set | O(1) avg | no sorted order†; hashing cost on huge keys; worst-case O(n) |
   | FIFO / both-ends ops | deque / ring buffer | O(1) | mid-access O(n) |
   | LIFO | array as stack | O(1) | — |
   | Repeated min/max | binary heap | O(log n) push/pop | no efficient arbitrary lookup/delete‡ |
   | Ordered iteration + insert + range queries | balanced BST / skip list (`SortedList`, `TreeMap`) | O(log n) | not in Python/JS stdlib — know your library |
   | Prefix/string-key queries | trie | O(key length) | memory-hungry; a dict is fine until it isn't (`string-algorithm-toolkit`) |
   | Connectivity / grouping | union-find | ~O(1) amortized | union/find only — no un-merge |
   | Frequency counting | hash map of counts (`Counter`) | O(1) per event | top-k needs a heap on top (`heap-and-priority-patterns`) |

   †Python dicts preserve *insertion* order — not *sorted* order; the confusion causes real bugs.
   ‡Arbitrary deletion from a heap → lazy tombstones (skip on pop), not re-heapify.

3. **Composite structures for composite needs — the standard combos:** LRU cache = hash map + doubly-linked list (`OrderedDict` pre-builds it); "lookup by id AND iterate by score" = dict + sorted structure maintained together — wrap the pair in one class so they can't drift (two parallel structures updated in nine places WILL desynchronize); graph = dict-of-lists adjacency (`graph-traversal-patterns`); interval aggregation at scale = segment/interval trees (verify you truly need them first).

4. **Apply the reality corrections the textbook omits:** below n≈10³, arrays beat everything via cache locality (a linear scan of 50 items outruns the tree AND often the hash) — don't build machinery for small n; memory overhead is real (a Python dict of small ints costs ~10× the array; at n=10⁸ the "fast" structure may not fit — numpy/arrays win by fitting); hash costs scale with key size (hashing 1KB strings per lookup isn't meaningfully O(1) — intern or pre-hash).

5. **Check the concurrency and mutation constraints:** mutated-while-iterated (most languages: error or corruption — collect-then-mutate); shared across threads (the default structure is almost never thread-safe — locks, concurrent variants, or immutable+replace); persistence/undo needs favor immutable/persistent structures.

6. **Default simple, upgrade on evidence.** The honest priority: array/list → dict/set the moment lookup or membership appears → everything else only when the op mix demands it. Exotic structures carry a bug tax and a reader tax; a heap earns its place when "repeatedly need the min" is *in the requirements*, not in the imagination.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "It's a ranking, so it needs a sorted structure" | If the ops are 'batch-compute weekly, read top 10,' it's an array sorted once. The noun said 'ranking'; the op-mix said 'array.' Access pattern, not noun. |
| "A list is fine — n is small right now" | 'Right now' is how the in-list membership check meets n=10⁵ in production. If growth is plausible, the one-word change to set costs nothing today. |
| "I'll re-sort each iteration to get the min — it's simple" | O(n log n) per step where a heap does O(log n). Simplicity that multiplies per-iteration is complexity on layaway. |
| "The trie/interval tree is the RIGHT structure for this" | For 200 words or 40 intervals, the dict and the list win on constants, memory, and bug count. Right-at-scale and right-at-your-n are different structures (step 4). |
| "I'll keep the dict and list in sync manually — it's two lines" | It's two lines in nine call sites, and the tenth forgets. The wrapping class (step 3) makes desync structurally impossible. |
| "Heaps don't support delete, so I'll rebuild it each time" | Lazy tombstones (mark deleted, skip on pop) solve arbitrary-delete in O(1) amortized — the ‡ catch has a standard answer that isn't O(n) rebuilds. |

## Red Flags

- `x in some_list` inside a loop over anything sizable.
- `list.insert(0, …)` / `shift()` used as a queue.
- Parallel dict+list of the same items updated in multiple places, unencapsulated.
- A whole-collection sort inside an iteration to extract one extreme.
- Exotic structures (tries, segment trees) at n where a scan is invisible in the profile.
- Structure chosen with no op-mix written anywhere.

## Verification

- [ ] Op-mix written (operations × frequency × n) — in the PR or design note.
- [ ] Chosen structure justified against the table row it matches — one line.
- [ ] Small-n check: no machinery below n≈10³ without a measured reason.
- [ ] Composite needs encapsulated in one type — no free-floating parallel structures in the diff.
- [ ] Mutation/concurrency constraints addressed where shared state exists — noted.
- [ ] For perf-motivated changes: before/after measurement at realistic n — numbers attached.

## Example

Task: "match incoming orders against the best open quote; quotes arrive, expire, and get cancelled; ~50k live quotes." Op mix written down: insert quote (frequent), pop best-price (frequent), cancel by id (frequent), expire by time (periodic). Best-price → heap keyed on price; cancel-by-id against a heap → the ‡ catch: lazy deletion — a `cancelled: set[QuoteId]`, popped quotes checked and skipped. Expiry → second heap keyed on expiry time, same tombstone set. Wrapped as one `QuoteBook` class holding both heaps + set + by-id dict, so the four structures can't drift. Considered and rejected: a sorted container (O(log n) for everything — cleaner, but the stdlib lacked one and two-heaps-plus-tombstones is boring and testable). Load test at 10× volume: pop p99 = 40µs. The alternative first draft — re-sorting a list per match — had been measured at 80ms per op: a 2000× difference from choosing by access pattern.

## Related skills

- `complexity-analysis` — verifying the chosen structure's bound survives the input.
- `heap-and-priority-patterns` — the heap rows of the table, expanded.
- `graph-traversal-patterns` — adjacency representations and their traversals.
- `string-algorithm-toolkit` — the trie row, expanded.
