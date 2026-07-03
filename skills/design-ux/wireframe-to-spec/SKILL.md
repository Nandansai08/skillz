---
name: wireframe-to-spec
description: >
  Use when turning wireframes/mockups into buildable specifications —
  annotating states, behaviors, edge cases, and data rules so engineering
  builds what design meant. Triggers: "spec this design", "annotate these
  wireframes", "handoff to engineering", "design spec", "what happens when"
  questions piling up mid-build.
---

# Wireframe to Spec

## When to use this skill
- A design is "done" visually and engineering is about to build it — the gap between the picture and the program needs writing down.
- Mid-build "what happens when...?" questions are arriving daily (the spec is being written live, expensively).
- NOT for the PRD layer (what/why — `prd-writing`) or the visual design itself; this is the behavioral contract between a finished design and its implementation.

## Prerequisites
- The wireframes/mocks at near-final fidelity, and access to the designer's intent (specs written by guessing at mocks reproduce the guessing in code).
- The data model behind the screens (field types, lengths, cardinalities — half the spec's edge cases come from data reality).

## Workflow

1. **Walk every element and ask the four questions:** what is it bound to (which field/query)? What can the user do to it? What are ALL its states? What happens at its extremes? A mock shows one frame of an interactive system; the spec is the other frames. Annotate ON the design (Figma comments/spec layers) so picture and contract can't drift apart.

2. **Attach the state matrix per component** — this is the spec's core cargo (`empty-loading-error-states` step 1's matrix, made deliverable): empty / loading / error / partial / ideal per data-bound element, plus interactive states (hover, focus, active, disabled-with-reason, selected) per control. Disabled states especially: WHY disabled, and how the user learns why (tooltip? adjacent text?) — the mystery-disabled-button is a spec omission wearing a UI.

3. **Specify the data rules the mock silently assumed:** truncation and overflow per text field (ellipsis at what width? full value on hover? wrap?), the long-value stress cases (the 60-character name in the 20-character slot — `edge-case-enumeration` step 2 applied to layout), number/date/currency formatting (locale? timezone? — "Jul 3" in the mock is a decision about neither), sort orders and their ties, pagination/virtualization thresholds, and max-plausible cardinalities (the dropdown mocked with 4 options that production feeds 400 — dropdown becomes typeahead at what count?).

4. **Write behaviors as event → outcome contracts:** for every interaction — click/submit/drag — the outcome chain including timing ("Save: button enters loading state, inputs lock, success → toast + return to list; failure → inline error per `form-design` step 4, inputs unlock, values preserved"). Include the impatience cases: double-click on submit (debounce? idempotent? — `api-design-rest` step 5's twin), navigation-away mid-save, concurrent edit by another user (last-write-wins? conflict prompt? — a product decision that will otherwise be made silently by whoever writes the mutation).

5. **Bind the spec to system vocabulary, not raw values:** colors/spacing/type referenced as tokens (`design-system-tokens` — "bg: `color-bg-danger`", never "#e5484d"), components by their library names with variant props, copy strings from the term sheet (`microcopy-writing` step 6) with final text — "lorem ipsum" and "TBD copy" in a handed-off spec are deferred decisions that will be made by an engineer at 6pm. Responsive behavior per `responsive-layout-strategy`: what reorganizes at each content breakpoint, specified as priority decisions, not left to compression.

6. **Include the a11y contract per component:** focus order and focus behavior on dynamic changes (modal open/close, deletion), announcement requirements for async results (`accessibility-audit` steps 2–3, specified rather than audited later), keyboard interactions for custom widgets, alt/label text as actual strings. Retrofit costs 5–10× (the audit skill's math); the spec is where a11y is cheapest.

7. **Review the spec as a dialogue, then track deltas:** a walkthrough with the implementing engineers BEFORE build starts — their "what about...?" questions found now are spec fixes, found in week 3 are rework (the walkthrough routinely finds 10–20 gaps; that's the mechanism working); during build, decisions made in-flight get written BACK into the spec (the spec at ship should describe what shipped — a divergent spec poisons the next feature that copies from it); a lightweight "deviations" log distinguishes agreed-changes from drift.

## Common pitfalls
- The happy-path handoff: pixel-perfect ideal states, zero coverage of empty/error/long-data — engineering invents the missing 80% ad hoc, and the design review afterward relitigates every invention (step 2's matrix is the prevention).
- Spec-by-redlines only: spacing and colors annotated exhaustively, behaviors not at all — the measurable specified, the meaningful omitted. Tokens make redlines nearly free (step 5); behaviors are where the writing effort belongs.
- Mock-data flattery: every name 8 characters, every list 4 items, every number round — the spec inherits the mock's optimism and the German locale destroys the layout in week one (step 3's stress cases).
- Silent product decisions delegated to implementation: concurrent edits, double-submits, mid-save navigation — each "edge case" is a policy choice someone with product context should make, on purpose (step 4).
- The frozen spec: build-time decisions never written back, so the document describes a product that doesn't exist — and the next team copies from it (step 7's write-back rule).
- Spec as substitute for conversation: 40 pages thrown over the wall, walkthrough skipped — the questions arrive anyway, mid-build, with rework attached.

## Example
Feature: team-members management screen (list, invite, role change, removal). Mock was clean; the spec pass generated the real contract: state matrix caught 9 undesigned cells (incl. "invite pending" — a whole state the mock lacked, needing its own row treatment and resend action); data rules pass surfaced that emails could be 90 chars (truncation rule written), roles list was 3 today but plugin-extensible (dropdown specified with overflow behavior at >6), and the members table needed a decision at >200 rows (virtualize). Behavior contracts caught the two silent product decisions: removing yourself (blocked with explanation — decided by PM in the spec review, not by an engineer alone later) and concurrent role-edit (last-write-wins + toast, decided likewise). A11y contract specified focus-return after the removal dialog and the live-region for invite results. The pre-build walkthrough found 6 more gaps in 40 minutes (cheapest 40 minutes of the project — the "what does the empty search state say?" question got a term-sheet answer instead of an invention). Build ran with FOUR mid-build questions total; the previous comparable feature had generated thirty-one, each a half-day round trip.

## Related skills
- `empty-loading-error-states` — the state matrix this spec carries.
- `design-system-tokens` + `microcopy-writing` — the vocabularies it binds to.
- `accessibility-audit` — specified here, audited there.
- `prd-writing` — the layer above; this spec implements that document's whats.
