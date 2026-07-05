---
name: note-synthesis
description: >
  Use when turning accumulated notes into structured output — converting
  reading/meeting/research notes into claims, connecting them, and
  building an outline that argues instead of lists. Triggers: "synthesize
  my notes", "turn these notes into a doc", "I've read everything and
  can't start writing", "organize this research", "notes to outline".
  NOT for multi-interview user research (see user-interview-synthesis)
  or live capture — this starts when capture ends.
---

# Note Synthesis

## Overview

"I've researched for weeks and can't write the first paragraph" is a synthesis gap, not a writing gap: notes are records of encounter; claims are statements with truth-value, and only claims can be outlined. The transformation — notes → claims → clusters → claim-shaped outline — is where the thinking happens; the writing afterward is transcription.

## When to Use

- Notes have accumulated (reading, meetings, investigations) and must become a document, decision, or talk.
- The can't-start-writing blockage.

**When NOT to use:**
- Multi-interview research — `user-interview-synthesis` (same spine, specialized).
- Live capture — this starts when capture ends.

## Prerequisites

- The notes, findable in one place — and the output's purpose named (the purpose is the relevance filter).

## The Workflow

1. **Convert notes to claims — the transformation everything depends on.** Raw notes record encounters ("read the replication docs"; "Meeting: Dana worried about failover"); claims have truth-value ("logical replication doesn't replicate DDL — schema changes need out-of-band coordination"; "failover confidence is low because it's never been drilled"). Walk the notes, extract every claim the purpose might need, one per line, each with its source pointer. The blockage is nearly always here: trying to outline encounters instead of claims.

2. **Keep the two layers distinguishable — evidence vs interpretation:** sourced claims ("the SLA excludes regional outages — contract §4.2") vs your inferences ("therefore multi-region is our responsibility"), marked differently. The discipline lets a reviewer — or you in a month — audit which parts are observation and which are reasoning atop it. Inference stacked on unmarked inference is how confident nonsense gets written.

3. **Cluster claims by affinity and name what emerges:** group claims bearing on the same tension, name clusters AFTER they form (pre-made buckets find only themselves), and let the leftovers pile exist honestly — misfit claims are either out-of-scope (park them) or the seed of the theme you haven't recognized yet; check which before discarding.

4. **Hunt the connections ACROSS clusters — where synthesis pays:** contradictions (each is a resolvable scope difference, a genuinely open question for the doc to name, or a mistake in your notes — all three are findings); dependencies ("claim C only matters if A holds"); and the recurring deep pattern (the same tension appearing in three clusters under different names is usually the document's actual thesis).

5. **Derive the outline from the claim-structure, shaped by the purpose:** clusters and their relations ARE the skeleton — a decision memo orders them situation → options → recommendation; a design doc problem-claims → constraints → design. The test: every section is a claim (an assertion the section will support), not a topic label — "Failover is our biggest unpracticed risk" writes itself; "Failover" invites another week of stalling.

6. **Write from claims, cite from pointers, notice the gaps:** drafting against a claim-outline is transcription-plus-connective-tissue; pointers become citations (`citation-management`); sections that resist writing are diagnostic — a missing claim (go get it) or a claim you don't believe (surface THAT — it's the doc's most honest sentence trying to get out).

7. **Archive the claim-base, not just the deliverable:** the document ships; the claim collection (sources, marks, parked leftovers) is the reusable asset — next quarter's adjacent question starts populated. Light structure (one greppable file per investigation) beats an elaborate system capture-discipline can't sustain.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "I'll re-read my highlights until the structure emerges" | Highlight re-reading is acquisition cosplaying as synthesis — encounters curated, claims never extracted. Structure emerges from claims; it never emerges from re-reading. |
| "Topic headings first, fill in the sections after" | Noun headings ('Failover') make every section a swamp; claim headings ('Failover is our biggest unpracticed risk') write themselves. The outline's shape decides whether drafting is transcription or stalling. |
| "I remember which parts were my speculation" | You won't: week-five you re-encounters week-two's guess as established fact. Memory launders inference into evidence; the marks (step 2) are the anti-laundering device. |
| "Every claim fits somewhere if I think hard enough" | Forcing the misfits buries the one that was the real finding — the example's crux claim had no cluster precisely because it was the unrecognized theme. The honest leftovers pile is where theses hide. |
| "These two sources conflict — I'll go with the stronger one" | Silently dropping one side edits out the open question the document OWED its readers. Contradictions are cargo: resolve with the scope difference, or name them. |
| "Synthesis happens when I write — capture now, think later" | Three weeks of capture + a Friday deadline = the panicked topic-outline swamp. Claim extraction in passes as material accumulates is what makes the final synthesis a day instead of a crisis. |

## Red Flags

- Weeks of notes, zero claims extracted.
- An outline of noun headings.
- No evidence/inference marking anywhere.
- Contradictions absent from a multi-source investigation (they were smoothed, not absent).
- The leftovers forced into clusters for tidiness.
- The claim file discarded after the doc ships.

## Verification

- [ ] Claim file exists: one claim per line, source pointers, evidence/inference marks — linked.
- [ ] Clusters named post-formation; leftovers pile visible with dispositions.
- [ ] Cross-cluster pass done: contradictions and dependencies listed; the recurring pattern named (or its absence noted).
- [ ] Outline sections are claim-sentences — every heading asserts something.
- [ ] Resistant sections diagnosed (missing claim fetched, or disbelief surfaced) — noted.
- [ ] Claim-base archived greppably for reuse — location linked.

## Example

Input: five weeks of migration investigation — 60 pages of notes across vendor docs, 9 meetings, two spikes, a ticket dump — due to become a go/no-go memo, author fully blocked. Claim extraction: 3 hours, 84 claims with pointers, evidence/inference marked. Clustering yielded five themes; the leftovers pile held four claims — three parked, one turned out to be the memo's crux (a licensing clause from meeting six that quietly capped the cost case — it had no cluster because nobody had connected it to the cost model yet). Cross-cluster hunt found one hard contradiction (vendor's stated throughput vs the spike's measurement — resolved as a scope difference: their number assumed batching the workload couldn't use; became the memo's key risk section) and the recurring pattern (three clusters reducing to "our confidence is simulation-based, not drill-based"). Outline: six claim-sentences. Draft: one day — after five blocked ones. The memo's recommendation (conditional go, licensing renegotiated first, one drill before cutover) traced every load-bearing sentence to a pointer; the claim file answered two exec follow-ups in minutes and seeded the Q3 renewal analysis without a single note re-read.

## Related skills

- `user-interview-synthesis` — the specialized sibling for interview data.
- `literature-review` — the matrix version for published sources.
- `citation-management` — the pointer infrastructure underneath.
- `research-question-framing` — the purpose that filters the claims.
