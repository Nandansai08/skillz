---
name: user-story-slicing
description: >
  Use when breaking work into stories — slicing vertically so each story
  ships user-visible value, applying INVEST, spotting stories too big to
  estimate. Triggers: "break this epic down", "slice these stories",
  "this ticket is too big", "vertical slicing", "story is unestimatable",
  "how do we split this feature". NOT for pure-technical work chunking
  (migrations, refactors decompose by risk and dependency).
---

# User Story Slicing

## Overview

A good slice is a thin end-to-end path — shallow through every layer, complete to a user-visible outcome. Horizontal layers ("build the API, then the UI, then integrate") defer all value and hide all integration risk in the last story; the split catalog and the 1–3-day rule are the antidotes.

## When to Use

- An epic/feature needs decomposition into shippable increments.
- A story has sat "in progress" for two weeks or draws estimates ranging 2-to-13.

**When NOT to use:**
- Pure-technical chunking (migrations, refactors) — decompose by risk/dependency (`refactor-safely` step 3), though deliver-value-early transfers.

## Prerequisites

- The feature's user outcomes (`prd-writing`), and agreement on what "done" includes (tested? deployed? flagged?).

## The Workflow

1. **Slice vertically: every story crosses the whole stack to a user-visible outcome.** The anti-pattern is horizontal layers — no layer is testable with real users, value arrives only when ALL land, and the integration story absorbs every deferred risk. A good slice: "user can upload a CSV and see row count" touches upload UI, parsing, storage — shallowly, completely.

2. **Find the seams with the standard split catalog** (try in order until one yields):
   - **By workflow step:** upload → validate → map → import → review errors.
   - **By business-rule subset:** flat shipping first, weight-based later.
   - **By data variation:** CSV before Excel; one currency first.
   - **By happy-path-first:** success path ships; error cases follow — EXCEPT error cases that lose data or money, which ride with the happy path (`error-handling-strategy` decides which).
   - **By operation:** view before edit; create before delete.
   - **By user segment:** admins first.
   - **Spike split** when uncertainty dominates: a timeboxed learning story with a QUESTION and knowledge as the deliverable.

3. **Check each slice against INVEST, weaponizing the two that catch real problems:** *Independent* (schedulable in any order — inter-story dependencies are queue poison; each one found is a re-slicing prompt, not a scheduling note) and *Valuable* (someone can see/verify something — "as a developer, I want a repository layer" fails and is a task inside a real story).

4. **Size rule: 1–3 days of one person's work including tests and review.** Bigger → back to the catalog. The unestimatable tell: estimates diverging widely in planning — that divergence is *unsurfaced disagreement about scope or approach*; the fix is conversation + a spike, never averaging the numbers.

5. **Write each story so the slice's boundary is unambiguous:** user-outcome title, acceptance criteria listing what's IN, and the "not in this story" line pointing at siblings ("errors beyond count-mismatch: story #4413") — the story-level non-goals that prevent slice bleed during implementation.

6. **Sequence by risk and learning, not architecture convenience:** first stories retire the most uncertainty (the gnarly integration, the performance question). "Walking skeleton" first — the thinnest full path on production infrastructure — then flesh per slice. The tempting order (easy CRUD first, scary integration last) discovers the project-killer in week 7 instead of week 1.

7. **Recombine when slicing overshoots:** ten 2-hour stories carry more ceremony than value. The grain target: "each story could ship, and shipping it would tell us or give users something" — both halves matter.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "Backend first, frontend after — that's how the work divides" | That's how the TEAM divides, not the value. Neither half is user-testable, and the 'integrate' story at the end inherits every deferred risk with none of the schedule left. |
| "As a developer, I want a service layer — it's still a story" | The persona is fake and the value is deferred; it's a task inside some real story. Valuable means someone outside the codebase can verify something happened. |
| "Split the 13-pointer into two 6.5s" | Estimate arithmetic without a seam produces coupled half-stories that can't ship independently and merge back in practice. The catalog finds seams; division finds fractions. |
| "Defer all error handling to the polish sprint" | Including the case where a failed import half-writes data? Risk-bearing edge cases ride with their happy path; 'polish' is where data-integrity bugs go to ship. |
| "The estimates are 2 and 8 — call it 5 and move on" | The divergence IS the information: two people are picturing different scopes. Averaging silences the signal; the conversation (or spike) resolves it in ten minutes. |
| "Do the easy CRUD stories first — build momentum" | Momentum toward the week-7 discovery that the scary integration doesn't work. Risk-first sequencing spends the schedule's cheapest days on the project's biggest unknowns. |

## Red Flags

- Stories named after layers or components instead of outcomes.
- An "integration" story at the end of every epic.
- Dependency chains: B needs A needs C on the board.
- Spikes with no timebox and no question.
- Stories in-progress past a week without a re-slicing conversation.
- Error handling universally deferred, including data-loss paths.

## Verification

- [ ] Every story states a user-visible outcome in its title — board reviewed.
- [ ] Each story's acceptance criteria + "not in this story" line present — spot-check three.
- [ ] No inter-story dependencies (or each one consciously accepted) — noted.
- [ ] All stories ≤3 days; estimate divergences resolved by conversation/spike, not averaging — planning notes.
- [ ] Walking skeleton scheduled first; the top uncertainty named and retired early — sequence shown.
- [ ] Risk-bearing error cases attached to their happy-path stories — checked against the error inventory.

## Example

Epic: "customers import their product catalog" (est. ~6 weeks, one scary unknown: real-world file chaos). Slicing session output: Story 1 — walking skeleton: upload one hardcoded-format CSV, rows land in DB, count shown (2 days, deployed behind a flag — retired deploy-pipeline and storage unknowns week 1). Story 2 — spike, timeboxed 2 days: run 30 real customer files through the parser, produce the actual-variation catalog (finding: 40% had BOM/encoding issues nobody predicted — reshaped every later story). Stories 3–7: column mapping UI; validation with per-row error report (rides with data-integrity errors — not deferred); encoding tolerance; Excel support; re-import/idempotency. Each story: 1–3 days, "not in this story" lines cross-referencing siblings. The estimate-divergence tell fired once — story 4 drew a 2 and an 8; the conversation surfaced that one engineer assumed streaming for huge files — resolved as an explicit non-goal ("files ≤50MB; bigger: error message"), story settled at 3.

## Related skills

- `prd-writing` — the source of outcomes and the non-goals discipline.
- `feature-scoping-cut` — cutting scope vs slicing it: the sibling decision.
- `prioritization-frameworks` — ordering the resulting backlog.
- `data-pipeline-idempotency` — the example's re-import story, grown up.
