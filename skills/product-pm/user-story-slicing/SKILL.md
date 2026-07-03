---
name: user-story-slicing
description: >
  Use when breaking work into stories — slicing vertically so each story
  ships user-visible value, applying INVEST, spotting stories too big to
  estimate. Triggers: "break this epic down", "slice these stories",
  "this ticket is too big", "vertical slicing", "story is unestimatable",
  "how do we split this feature".
---

# User Story Slicing

## When to use this skill
- An epic/feature needs decomposition into shippable increments.
- A story has sat "in progress" for two weeks or draws estimates ranging 2-to-13.
- NOT for pure-technical work chunking (migrations, refactors) — those decompose by risk and dependency (`refactor-safely` step 3), though the deliver-value-early instinct transfers.

## Prerequisites
- The feature's user outcomes (from the PRD or equivalent — `prd-writing`), and agreement on what "done" includes here (tested? deployed? flagged?).

## Workflow

1. **Slice vertically: every story crosses the whole stack to a user-visible outcome.** The anti-pattern is horizontal layers ("build the API", "build the UI", "integrate") — no layer is testable with real users, value arrives only when ALL land, and the integration story absorbs every deferred risk. A good slice is a thin end-to-end path: "user can upload a CSV and see row count" touches upload UI, parsing, storage — shallowly, completely.

2. **Find the seams with the standard split catalog** (try in order until one yields):
   - **By workflow step:** upload → validate → map columns → import → review errors (each step usable, later steps stubbed with honest messaging).
   - **By business-rule subset:** flat shipping first, weight-based later; one tax jurisdiction, then the matrix.
   - **By data variation:** CSV before Excel; one currency before many.
   - **By happy-path-first:** the success path ships; each error/edge case is its own story (with the *dangerous* caveat: error cases that lose data or money are not deferrable — they ride with the happy path; `error-handling-strategy` decides which).
   - **By operation:** view before edit; create before delete.
   - **By user segment:** admins first, then members.
   - **Spike split** when uncertainty dominates: a timeboxed learning story ("prototype the parser against 20 real files, 2 days") whose output is knowledge that makes the rest sliceable.

3. **Check each slice against INVEST, weaponizing the two that catch real problems:** *Independent* (schedulable in any order? — dependencies between stories are queue poison) and *Valuable* (a user or stakeholder can see/verify something — "as a developer, I want a repository layer" fails this and is a task inside some real story, not a story). Negotiable, Estimable, Small, Testable follow mostly from vertical + the size rule below.

4. **Size rule: 1–3 days of one person's work including tests and review.** Bigger → return to step 2's catalog. The tell-tale unestimatable story: estimates diverge widely in planning — that divergence is *unsurfaced disagreement about scope or approach*, and the fix is conversation + a spike, never averaging the estimates.

5. **Write each story so the slice's boundary is unambiguous:** the user-outcome title, acceptance criteria listing what's IN, and — as important — an explicit "not in this story" line pointing to the sibling stories ("errors beyond count-mismatch: story #4413"). This is the story-level version of `prd-writing`'s non-goals, and it's what prevents slice bleed during implementation.

6. **Sequence by risk and learning, not by architecture convenience:** first stories = the ones that retire the most uncertainty (the gnarliest integration, the performance question) and generate real feedback. "Walking skeleton" first — the thinnest full path deployed to production infrastructure — then flesh per slice. The tempting order (all the easy CRUD first, scary integration last) discovers the project-killing problem in week 7 instead of week 1.

7. **Recombine when slicing overshoots:** ten 2-hour stories carry more ceremony than value — merge slices whose separation nobody benefits from. The grain target is "each story could ship, and shipping it would tell us or give users something" — both halves matter.

## Common pitfalls
- Horizontal slices wearing story format ("As a developer, I want the database schema...") — the persona is fake, the value is deferred, the integration risk is hidden. Vertical or it's a task.
- Deferring ALL error handling to "polish" stories — including the case where a failed import half-writes data. Risk-bearing edge cases ride with their happy path (step 2's caveat).
- Slicing by estimate arithmetic ("split the 13 into two 6.5s") without a seam — two coupled half-stories that can't ship independently and merge back in practice.
- The dependency chain: story B needs A needs C — one delay cascades. Each dependency found in step 3 is a re-slicing prompt, not a scheduling note.
- Spike stories without a timebox or a question — "investigate parsing" becomes a week of tourism. Spikes have a question, a timebox, and knowledge as the deliverable.
- Demanding every story be demo-worthy to *external* users — internal-user value, ops value, and validated-learning value count; the walking skeleton's "value" is retired deployment risk.

## Example
Epic: "customers import their product catalog" (est. ~6 weeks, one scary unknown: real-world file chaos). Slicing session output: Story 1 — walking skeleton: upload one hardcoded-format CSV, rows land in DB, count shown (2 days, deployed behind a flag — retired deploy-pipeline and storage unknowns week 1). Story 2 — spike, timeboxed 2 days: run 30 real customer files through the parser, produce the actual-variation catalog (finding: 40% had BOM/encoding issues nobody predicted — reshaped every later story). Stories 3–7: column mapping UI; validation with per-row error report (rides with data-integrity errors — not deferred); encoding tolerance (directly from spike findings); Excel support; re-import/idempotency (`data-pipeline-idempotency` at product scale). Each story: 1–3 days, "not in this story" lines cross-referencing siblings. The estimate-divergence tell fired once — story 4 drew a 2 and an 8; the conversation surfaced that one engineer assumed streaming for huge files — resolved as an explicit non-goal ("files ≤50MB; bigger: error message + sales ping"), story settled at 3.

## Related skills
- `prd-writing` — the source of outcomes and the non-goals discipline.
- `feature-scoping-cut` — cutting scope vs slicing it: the sibling decision.
- `prioritization-frameworks` — ordering the resulting backlog.
- `data-pipeline-idempotency` — the example's re-import story, grown up.
