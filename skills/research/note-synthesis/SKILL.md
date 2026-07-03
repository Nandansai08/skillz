---
name: note-synthesis
description: >
  Use when turning accumulated notes into structured output — converting
  reading/meeting/research notes into claims, connecting them, and building
  an outline that argues instead of lists. Triggers: "synthesize my notes",
  "turn these notes into a doc", "I've read everything and can't start
  writing", "organize this research", "notes to outline".
---

# Note Synthesis

## When to use this skill
- Notes have accumulated (reading, meetings, investigations) and must become a document, decision, or talk.
- The "I've researched for weeks and can't write the first paragraph" blockage — almost always a synthesis gap, not a writing gap.
- NOT for multi-interview user research specifically (`user-interview-synthesis` — same spine, specialized) or the live capture of notes (this skill starts when capture ends).

## Prerequisites
- The notes, findable in one place — and the output's purpose named (a decision memo, a design doc, a talk — the purpose is the relevance filter, same as `research-question-framing`'s question).

## Workflow

1. **Convert notes to claims — the transformation everything else depends on:** raw notes are records of encounter ("read the Postgres replication docs", "Meeting: Dana worried about failover"); claims are statements with truth-value ("logical replication doesn't replicate DDL — schema changes need out-of-band coordination"; "failover confidence is low because it's never been drilled"). Walk the notes and extract every claim your purpose might need, one per line/card, each carrying its source pointer. The blockage in the "can't start writing" state is nearly always this: trying to outline encounters instead of claims.

2. **Keep the two layers distinguishable — evidence vs interpretation:** claims sourced from material ("the vendor's SLA excludes regional outages — contract §4.2") vs claims you're inferring ("therefore multi-region is our responsibility") get marked differently. The discipline (shared with `user-interview-synthesis` step 1) is what lets a reviewer — or you in a month — audit which parts are load-bearing observation and which are your reasoning atop it. Inference stacked on unmarked inference is how confident nonsense gets written.

3. **Cluster claims by affinity and name what emerges:** group claims that bear on the same tension or theme, name each cluster AFTER it forms (pre-imposed buckets find only themselves — bottom-up, per the interview skill's same rule), and let the leftovers pile exist honestly (claims that fit nowhere are either out-of-scope for this purpose — park them — or the seed of the theme you haven't recognized yet; check which before discarding).

4. **Hunt the connections ACROSS clusters — where synthesis pays:** contradictions between claims (two sources, two meetings, or a source vs your own inference — each contradiction is either a resolvable scope difference, a genuinely open question for the doc to name, or a mistake in your notes: all three are findings); dependencies ("claim C only matters if claim A holds"); and the recurring deep pattern (the same tension appearing in three clusters under different names is usually the document's actual thesis). Linking passes like this are what Zettelkasten-style systems mechanize — the value is the traversal, whatever the tooling.

5. **Derive the outline from the claim-structure, shaped by the purpose:** the clusters and their relations ARE the skeleton — a decision memo orders them as situation → options (each backed by its claims) → recommendation; a design doc as problem-claims → constraint-claims → design-responding-to-them; a talk as tension → resolution. The test that the outline works: every section is a claim (an assertion the section will support), not a topic label — "Failover is our biggest unpracticed risk" writes itself; "Failover" invites another week of stalling.

6. **Write from claims, cite from pointers, notice the gaps:** drafting against a claim-outline is transcription-plus-connective-tissue (the hard thinking already happened in steps 1–4); the source pointers become the citations (`citation-management` holds them); and the sections that resist writing are diagnostic — a section that won't flow usually has a missing claim (go get it: one more source, one more measurement) or a claim you don't actually believe (surface THAT — it's the doc's most honest sentence trying to get out).

7. **Archive the claim-base, not just the deliverable:** the document ships; the claim collection (with sources, marks, and the parked leftovers) is the reusable asset — next quarter's adjacent question starts from a populated base instead of re-reading everything (`literature-review` step 7's same economics). A light structure (one file/folder per investigation, claims greppable) beats an elaborate system that capture-discipline can't sustain.

## Common pitfalls
- Outlining from topics instead of claims: the section headers are nouns, every section is a swamp, and the writing stalls exactly as before — the step-1 transformation was skipped.
- Highlight hoarding as synthesis: 200 highlights re-read, re-sorted, re-colored — encounters curated, claims never extracted. Highlighting is acquisition; synthesis starts at the claim.
- Unmarked inference contamination: your speculation from meeting three, re-encountered in week five as an established fact of the project (step 2's marks exist because memory launders).
- Forcing the leftovers: every claim crammed into some cluster for tidiness, burying the misfit that was the real finding (step 3's honest pile).
- Contradiction-smoothing: two conflicting claims silently reconciled by dropping one — the open question the document OWED its readers, edited out for flow (step 4 treats contradictions as cargo, not noise).
- Synthesis postponed to writing time: three weeks of capture, then "the doc is due Friday" — steps 1–4 compressed into a panicked evening produce the topic-outline swamp. Synthesize in passes as material accumulates; the claim file grows alongside the notes.

## Example
Input: five weeks of migration investigation — 60 pages of notes across vendor docs, 9 meetings, two spikes, a support-ticket dump — due to become a go/no-go memo, author fully blocked ("I know everything and can't write anything"). Claim extraction: 3 hours, 84 claims with source pointers, evidence/inference marked. Clustering yielded five themes; the leftovers pile held four claims that didn't fit — three parked, one turned out to be the memo's crux (a licensing clause from meeting six that quietly capped the migration's cost case — it had no cluster because nobody had connected it to the cost model yet: step 3's "unrecognized theme" case, live). Cross-cluster hunt found one hard contradiction (the vendor's stated throughput vs the spike's measurement — resolved as a scope difference: their number assumed batching the workload couldn't use; became the memo's key risk section) and the recurring pattern (three clusters all reduced to "our confidence is simulation-based, not drill-based"). Outline: six claim-sentences. Draft: one day — after five blocked ones. The memo's recommendation (conditional go, licensing clause renegotiated first, one drill before cutover) traced every load-bearing sentence to a pointer; the claim file answered two follow-up questions from the exec review in minutes, and seeded the Q3 vendor-renewal analysis without a single note re-read.

## Related skills
- `user-interview-synthesis` — the specialized sibling for interview data.
- `literature-review` — the matrix version for published sources.
- `citation-management` — the pointer infrastructure underneath.
- `research-question-framing` — the purpose that filters the claims.
