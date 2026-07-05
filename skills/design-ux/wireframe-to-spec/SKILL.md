---
name: wireframe-to-spec
description: >
  Use when turning wireframes/mockups into buildable specifications —
  annotating states, behaviors, edge cases, and data rules so engineering
  builds what design meant. Triggers: "spec this design", "annotate these
  wireframes", "handoff to engineering", "design spec", "what happens
  when" questions piling up mid-build. NOT for the PRD layer (see
  prd-writing) or the visual design itself — this is the behavioral
  contract between a finished design and its implementation.
---

# Wireframe to Spec

## Overview

A mock shows one frame of an interactive system; the spec is the other frames — states, data rules, behavior contracts, and the product decisions (concurrent edits, double-submits) that otherwise get made silently by whoever writes the mutation. The pre-build walkthrough finds ten gaps in forty minutes; mid-build, each gap is a half-day round trip.

## When to Use

- A design is "done" visually and engineering is about to build it.
- Mid-build "what happens when...?" questions arriving daily (the spec is being written live, expensively).

**When NOT to use:**
- The what/why layer — `prd-writing`.
- The visual design itself.

## Prerequisites

- Wireframes/mocks at near-final fidelity, and access to the designer's intent.
- The data model behind the screens (field types, lengths, cardinalities — half the edge cases come from data reality).

## The Workflow

1. **Walk every element and ask the four questions:** what is it bound to? What can the user do to it? What are ALL its states? What happens at its extremes? Annotate ON the design (Figma comments/spec layers) so picture and contract can't drift apart.

2. **Attach the state matrix per component** — the spec's core cargo (`empty-loading-error-states` step 1, made deliverable): empty/loading/error/partial/ideal per data-bound element, plus interactive states (hover, focus, disabled-with-reason, selected) per control. Disabled states especially: WHY disabled, and how the user learns why — the mystery-disabled-button is a spec omission wearing a UI.

3. **Specify the data rules the mock silently assumed:** truncation per text field (ellipsis at what width? full value on hover?), the long-value stress cases (the 60-char name in the 20-char slot — `edge-case-enumeration` applied to layout), number/date/currency formatting (locale? timezone? — "Jul 3" in the mock is a decision about neither), sort orders and ties, pagination thresholds, and max-plausible cardinalities (the dropdown mocked with 4 options that production feeds 400 — becomes a typeahead at what count?).

4. **Write behaviors as event → outcome contracts:** per interaction, the outcome chain including timing ("Save: button enters loading, inputs lock; success → toast + return to list; failure → inline error, inputs unlock, values preserved"). Include the impatience cases: double-click on submit (debounce? idempotent?), navigation-away mid-save, concurrent edit by another user (last-write-wins? conflict prompt?) — each a product decision that will otherwise be made silently by an engineer at 6pm.

5. **Bind the spec to system vocabulary, not raw values:** colors/spacing as tokens (`design-system-tokens` — "bg: `color-bg-danger`", never a hex), components by library names with variant props, copy as final strings from the term sheet (`microcopy-writing` — "lorem ipsum" in a handed-off spec is a deferred decision an engineer will make). Responsive behavior per `responsive-layout-strategy`: what reorganizes at each breakpoint, as priority decisions.

6. **Include the a11y contract per component:** focus order and focus behavior on dynamic changes (modal open/close, deletion), announcement requirements for async results, keyboard interactions for custom widgets, alt/label text as actual strings (`accessibility-audit` specified rather than audited later — retrofit costs 5–10×).

7. **Review the spec as a dialogue, then track deltas:** a walkthrough with the implementing engineers BEFORE build (their "what about...?" questions found now are spec fixes; found in week 3 they're rework — the walkthrough routinely finds 10–20 gaps, which is the mechanism working); during build, in-flight decisions get written BACK into the spec (the spec at ship describes what shipped — a divergent spec poisons the next feature that copies from it); a lightweight deviations log separates agreed-changes from drift.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "The mock is detailed — engineering can infer the rest" | The mock is one frame of an interactive system. Engineering infers the other frames ad hoc, and the design review afterward relitigates every inference — at rework prices. |
| "Redlines are done — spacing and colors all annotated" | The measurable got specified; the meaningful (behaviors, states, edge cases) didn't. Tokens make redlines nearly free; the writing effort belongs to the contracts. |
| "Concurrent edits are an engineering detail" | Last-write-wins vs conflict-prompt is a product decision about whose work gets destroyed. Silent delegation means it's decided by whoever writes the mutation, without the context. |
| "Real copy can come later — lorem ipsum for now" | 'Later' is an engineer at 6pm inventing the error message that ships. Final strings from the term sheet, in the spec, or the copy decision was delegated by accident. |
| "The mock's data looks realistic enough" | Every name 8 characters, every list 4 items, every number round — the mock's optimism becomes the spec's blind spot, and the German locale destroys the layout in week one. Stress cases are step 3's whole job. |
| "The spec is thorough — skip the walkthrough, start building" | Forty pages over the wall still yields the mid-build question parade. The walkthrough's 10–20 caught gaps in 40 minutes are the cheapest defects the project will ever fix. |

## Red Flags

- Mid-build "what happens when" questions arriving daily.
- Disabled controls with no specified reason or explanation path.
- Specs with hex values, placeholder copy, or screenshot-only annotations.
- No entry anywhere for double-submit, mid-save navigation, or concurrent edits.
- The shipped product diverging from a spec nobody updated.
- A11y absent from the spec, "to be audited later."

## Verification

- [ ] Four-questions walk done per element — annotations on the design, linked.
- [ ] State matrix complete per component (data states + interactive states, disabled-with-reason) — attached.
- [ ] Data rules specified: truncation, formats, cardinalities, stress cases — section present.
- [ ] Behavior contracts cover the impatience cases (double-submit, mid-save nav, concurrent edit) — each with a product decision recorded.
- [ ] Spec binds to tokens, component names, and final strings — zero hexes/lorem-ipsum.
- [ ] Pre-build walkthrough held; gap count and fixes noted; deviations log active through build.

## Example

Feature: team-members management screen (list, invite, role change, removal). Mock was clean; the spec pass generated the real contract: state matrix caught 9 undesigned cells (incl. "invite pending" — a whole state the mock lacked, needing its own row treatment and resend action); data rules surfaced that emails could be 90 chars (truncation rule written), roles were 3 today but plugin-extensible (dropdown specified with overflow behavior at >6), and the table needed a decision at >200 rows (virtualize). Behavior contracts caught the two silent product decisions: removing yourself (blocked with explanation — decided by the PM in spec review, not by an engineer alone later) and concurrent role-edit (last-write-wins + toast). A11y contract specified focus-return after the removal dialog and the live-region for invite results. The pre-build walkthrough found 6 more gaps in 40 minutes. Build ran with FOUR mid-build questions total; the previous comparable feature had generated thirty-one, each a half-day round trip.

## Related skills

- `empty-loading-error-states` — the state matrix this spec carries.
- `design-system-tokens` + `microcopy-writing` — the vocabularies it binds to.
- `accessibility-audit` — specified here, audited there.
- `prd-writing` — the layer above; this spec implements that document's whats.
