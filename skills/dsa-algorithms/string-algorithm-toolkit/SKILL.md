---
name: string-algorithm-toolkit
description: >
  Use when string problems outgrow brute force — substring search, prefix
  queries, string hashing, and knowing when naive is genuinely fine.
  Triggers: "substring search at scale", "find all occurrences",
  "autocomplete prefix matching", "string matching too slow", "rolling
  hash", "trie or dict". NOT for regex authoring, parsing/grammars, or
  fuzzy/semantic matching (edit distance DP or embeddings).
---

# String Algorithm Toolkit

## Overview

The toolkit's most-used tool is the baseline: your language's built-in find is C-optimized and wins below n≈10⁷ for single patterns. Escalation — Aho-Corasick, rolling hashes, tries — is justified by sizes written down (k patterns × n text), not by the algorithms being interesting.

## When to Use

- String matching/indexing where input size makes the naive approach a real cost — or where you suspect it might and want the threshold check.
- Choosing between dict, trie, hashing, and automaton approaches.

**When NOT to use:**
- Regex authoring, parsing/grammars — different toolset.
- Fuzzy/semantic matching — edit-distance DP or embeddings.

## Prerequisites

- Sizes written down: text length n, pattern length m, number of patterns k, alphabet. The whole skill is choosing by these numbers.

## The Workflow

1. **Start from the do-nothing baseline: your language's built-in find is excellent.** `str.find` / `in` / `indexOf` are C-optimized, often SIMD-accelerated, implementing Boyer-Moore-class algorithms already. For one pattern in one text, the built-in wins below n ≈ 10⁷ almost regardless of theory. Escalate only on: many patterns, many queries against one text, adversarial input, or streaming.

2. **One pattern, paranoia about worst cases → know what KMP buys.** Naive is O(nm) worst case but O(n) on random text; KMP guarantees O(n+m) with no backtracking (streaming-safe). In practice: stdlib until profiling or adversarial input (pathological repetition) proves otherwise; implement KMP's failure function only when you need its *byproduct* — the border array (longest proper prefix = suffix), which solves period-detection problems directly.

3. **Many patterns against one stream → Aho-Corasick.** k patterns matched simultaneously in O(n + total pattern length + matches): the content-filter / log-scanner shape (`pyahocorasick`, or trie + failure links). k separate `find` passes = O(kn) — the difference between 200 keyword scans per document and one.

4. **Many substring queries on one fixed text → rolling hash (or suffix structures at the extreme).** Polynomial rolling hash gives O(1) substring-equality after O(n) precompute:
   ```python
   # h[i] = hash of s[:i]; hash(s[l:r]) = (h[r] - h[l]*pow[r-l]) % M
   ```
   Rules that keep it honest: large prime modulus (or two — collision probability squared), random base per run (defeats crafted collisions — hashes on adversarial input are a security surface), and **verify candidate matches by direct comparison** when a false positive costs correctness. Suffix arrays/automata for "longest repeated substring"/"count distinct" — use a library; hand-rolling one under deadline is how weekends die.

5. **Prefix-shaped workloads → trie, but only past the dict threshold.** Below ~10⁴ keys, `bisect` on a sorted list or a dict is simpler and often faster (cache locality; tries chase pointers). The trie earns its memory when: prefix *enumeration* is the hot path, keys share long prefixes heavily, or per-node augmentation (counts, best-completion) is needed. `data-structure-selection` step 4's overhead math applies double.

6. **Normalize before matching, or match wrong:** Unicode NFC/NFD (é as one codepoint vs two — combining characters break naive equality), case-folding (`casefold()`, not `lower()`, for i18n), byte-vs-codepoint-vs-grapheme semantics decided explicitly. Every "search only fails for some users" bug report starts here.

7. **Close with the honesty check: state why the chosen tier is needed.** One sentence in the PR: "Aho-Corasick: 3k patterns × 1M docs/day makes per-pattern find O(kn) ≈ 10¹¹" — or equally valid: "naive scan: n ≤ 10⁴, hourly; stdlib suffices." Escalation without the sizes written down is résumé-driven engineering.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "I'll implement Boyer-Moore — faster than naive search" | The stdlib already implements better, in C, with SIMD. Hand-rolled classics are slower AND buggier than the built-in; measure before writing a single line. |
| "Equal hashes = equal strings, near enough" | At billion-comparison scale, the one-in-a-billion collision arrives on schedule. Verify-on-candidate (step 4) when false positives cost correctness; near-enough is a latent data bug. |
| "Fixed hash base is fine — who'd attack a string hash?" | Anyone whose input reaches your public surface: crafted collisions degrade Rabin-Karp to O(nm) and corrupt dedup. Random base per run costs one line. |
| "A trie is the correct structure for autocomplete" | For 500 words at 10 QPS, `startswith` in a loop is invisible in the profile and free of pointer-chasing machinery. Correct-at-scale ≠ correct-at-your-n (step 5's threshold). |
| "ASCII tests pass — matching works" | The 'search can't find café' bug ships exactly this way: composed vs decomposed input from different keyboards. NFC normalization before build AND scan (step 6), or the misses are silent. |
| "The clever matcher is fast — done" | While the O(n²) substring-generation loop next to it owns the runtime. The surrounding code's complexity swamps the algorithm's; profile the whole path. |

## Red Flags

- Hand-implemented KMP/Boyer-Moore in application code with no benchmark against stdlib.
- Rolling-hash equality treated as definitive on a correctness path.
- Fixed base+modulus hashing user-facing input.
- k patterns scanned in a k-iteration loop of `find` at production scale.
- Matching raw bytes on user text with no normalization pass.
- An escalated tier with no sizes-justification sentence anywhere.

## Verification

- [ ] Sizes written (n, m, k) with the tier choice derived from them — the step-7 sentence in the PR.
- [ ] Stdlib baseline measured before any custom implementation — numbers attached.
- [ ] Hash uses: random base + verify-on-candidate for correctness paths — code shown.
- [ ] Normalization applied consistently to both index and query sides — test with composed/decomposed pairs.
- [ ] Trie adoption justified past the dict threshold (key count + workload named).
- [ ] Before/after throughput at production scale — measured.

## Example

Real task: compliance scanner — 4,200 restricted phrases against ~800k support messages/day, currently 40 min/day via per-phrase `in` loops and growing linearly with the phrase list. Sizes: k=4200, n≈10⁹ chars/day → O(kn) ≈ 4×10¹² — the escalation is justified on paper before any code. Aho-Corasick automaton built once at startup (0.8s, 60MB), stream scanned in one pass: 92 seconds/day. Normalization pass added after week one: NFKC + casefold before both automaton build and scan — the missed matches in the pilot were decomposed accents from mobile keyboards, exactly the step-6 classic. Phrase list doubled the next quarter; scan time moved 92s → 96s. The rejected alternative (regex alternation of 4200 phrases) had been tried previously: the compiled pattern hit catastrophic size and the scan ran hours.

## Related skills

- `data-structure-selection` — the trie-vs-dict threshold logic.
- `complexity-analysis` — the sizes-first discipline this skill runs on.
- `edge-case-enumeration` — the Unicode cases that break matching silently.
- `two-pointer-sliding-window` — character-window problems that need no toolkit at all.
