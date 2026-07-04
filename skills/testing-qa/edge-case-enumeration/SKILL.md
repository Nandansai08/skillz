---
name: edge-case-enumeration
description: >
  Use when you need to systematically find the inputs that break code —
  boundaries, empties, nulls, encoding, time, concurrency — instead of
  guessing. Works for writing tests or reviewing them. Triggers: "what
  edge cases am I missing", "test cases for this function", "what could
  break this", "boundary conditions", "did I cover everything". NOT a
  substitute for property-based testing on large input spaces with
  statable invariants (see property-based-testing).
---

# Edge Case Enumeration

## Overview

Edge cases aren't discovered by staring harder — they're enumerated from per-type checklists that encode twenty years of production surprises. Walking the checklist converts "did I think of everything?" into a mechanical pass with a ranked output.

## When to Use

- Writing tests and wanting confidence the input space is covered.
- Reviewing a PR's tests for gaps.

**When NOT to use:**
- Large input spaces with statable invariants — `property-based-testing` generates what this skill enumerates.

## Prerequisites

- The function/endpoint signature and what each input means (units, valid ranges).

## The Workflow

Walk each input through the checklist for its type, then the cross-cutting categories. Write down every case that's *reachable*; strike ones validation upstream provably blocks.

1. **Numbers:** zero, negative, boundary value and boundary±1 (off-by-one lives here), max/min of the type, float specials (NaN, ±Inf, -0.0), values causing overflow in *intermediate* computation (`(a+b)/2`), precision traps (0.1+0.2), and huge-but-valid (1e15 cents).

2. **Strings:** empty, single char, whitespace-only, leading/trailing whitespace, very long (past any column/buffer limit), Unicode beyond ASCII (é, 中, emoji incl. multi-codepoint 👨‍👩‍👧), combining characters vs precomposed (é as one codepoint vs e+◌́), null byte, newline/tab embedded, case variants if compared, and strings that are *syntax* somewhere downstream: `'; --`, `<script>`, `{{}}`, `%s`, a filename like `../../etc/passwd`.

3. **Collections:** empty, one element, exactly the page/batch-size boundary and ±1, duplicates, all-identical elements, already-sorted and reverse-sorted (perf + logic), nested empties (`[[]]`), and max realistic size.

4. **Null/absent — distinguish the three:** missing key, key present with null, key present with empty value. APIs and DBs treat these differently; tests should too.

5. **Time:** midnight, end of month (Jan 31 + 1 month = ?), Feb 29, Dec 31→Jan 1, DST spring-forward gap (2:30am doesn't exist) and fall-back overlap (1:30am happens twice), epoch 0, pre-1970, timezone-naive meets timezone-aware, and "now" moving between two reads in the same function.

6. **State & sequence:** operation on uninitialized/closed/already-done state (cancel a cancelled order), repeated identical calls (idempotency), operations out of expected order, and the first-ever call (empty table, cold cache).

7. **Concurrency (when shared state exists):** two identical requests simultaneously (double-submit), read-modify-write interleaving, and the row deleted between your check and your use (TOCTOU).

8. **Rank and cut.** Score each case: likelihood × damage. Test everything user-reachable with real damage; drop cases upstream types/validation genuinely make impossible — and add one test *proving* the validation blocks them. Without this cut, the valuable cases drown in a slow 200-case suite.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "The UI prevents that input" | The API accepts what the form wouldn't send. Reachability is defined by the API surface, not the happy-path client. |
| "I tested the boundary — 100 works" | The off-by-one lives at 99 and 101. Boundary alone passing says almost nothing; the ±1 pair is the actual probe. |
| "Unicode is handled — I tested 'José'" | Multi-codepoint emoji and combining characters are what break length checks and slicing. One accented Latin char tests the shallow end. |
| "These values are impossible in practice" | Then one test proving the validation blocks them costs a minute and converts an assumption into a guarantee. "Impossible" without the proof is a production incident's opening line. |
| "Enumerating all this is over-testing" | Enumerating is cheap; TESTING everything would be over-testing — which is what step 8's rank-and-cut exists to prevent. The list costs ten minutes; the ranking spends wisely. |
| "Concurrency cases can't be unit-tested anyway" | Double-submit and TOCTOU reproduce fine with two interleaved calls in a test. Hard ≠ impossible, and these are the cases with incident-shaped damage. |

## Red Flags

- A test suite for a numeric function with no zero, negative, or boundary±1 cases.
- String handling tested exclusively with ASCII.
- "N/A" as the concurrency answer for code that touches shared rows.
- Every enumerated case tested with equal priority — 200 tests, no ranking, 40-second suite.
- The three null variants (missing / null / empty) collapsed into one test.
- No test proving upstream validation actually blocks the "impossible" inputs.

## Verification

- [ ] Checklist walked per input; the enumerated list exists (PR notes or test-plan comment).
- [ ] Boundary±1 pairs present for every numeric boundary in the contract.
- [ ] Struck cases each carry a reason ("blocked by schema X") and the blocking is itself tested once.
- [ ] Ranking applied: high likelihood×damage cases all tested; the cut documented.
- [ ] For shared-state code: double-submit and check-then-act cases present or explicitly dispositioned.

## Example

`paginate(items, page, per_page)` — enumeration found: `page=0` (is it 1-indexed?), negative page, page past the end, `per_page=0` (division), empty items, items length exactly `per_page` (does page 2 exist as empty or 404?), and `per_page=10**9` (allocation). Eight tests; two found real bugs: `page=0` returned the last page via Python negative indexing, and length-equals-per_page produced a phantom empty page 2 that broke the mobile client's infinite scroll.

## Related skills

- `unit-test-design` — the structure these cases get written into.
- `property-based-testing` — generating cases from invariants instead of checklists.
- `input-validation-boundaries` — deciding where these cases get *blocked* rather than handled.
