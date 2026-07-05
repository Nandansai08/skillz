---
name: form-design
description: >
  Use when designing or fixing forms — field ordering, validation timing,
  error copy placement, mobile input types, reducing abandonment.
  Triggers: "design this form", "form abandonment is high", "validation
  UX", "checkout form", "signup form review", "too many form errors".
  NOT for the backend validation rules themselves (see
  input-validation-boundaries) — this skill owns how humans experience
  them.
---

# Form Design

## Overview

The best form-design tool is deletion, the second is field-level instrumentation — which converts "the form underperforms" into "the VAT field is where 30% die." Validation timing follows reward-early-punish-late, and the input layer's cheap wins (autocomplete, inputmode, format tolerance) outconvert most redesigns.

## When to Use

- Building any form beyond trivial (signup, checkout, settings, applications).
- A form's completion/abandonment metrics are bad and need diagnosis.

**When NOT to use:**
- Server-side validation architecture — `input-validation-boundaries` (the two must agree on the rules).

## Prerequisites

- The list of fields with each one's justification — and the authority to challenge it.

## The Workflow

1. **Delete fields before designing them:** per field — needed NOW (vs collectable post-conversion)? Derivable (city from postal code)? Merely nice-for-marketing? Every removed field is measurable abandonment recovered; the survivor must name its consumer. Mark the *minority* (required or optional, whichever is fewer).

2. **Order and group by the user's mental flow:** logical sections matching how users think (who you are → where to ship → how to pay), easy-before-invasive (email before card; phone last with the "why we need this" line), single column (multi-column forms cause skipped fields — the research is unusually unanimous), labels *above* fields — not placeholder-as-label: it vanishes on focus, fails recall mid-fill, and fails `accessibility-audit`'s announcement check.

3. **Time validation by reward-early, punish-late:** validate on blur-after-input (not per-keystroke while typing — premature red is hostile), EXCEPT flip to instant-positive once valid (the checkmark as they finish). Never "invalid email" at `jo`; never save everything for submit (the scroll-back error hunt is the abandonment machine). Async checks show pending state inline and announce it.

4. **Write errors as fix-instructions, placed at the field:** what's wrong + how to fix, human words, next to the field, state marked by color + icon + text (never color alone). "Card declined — check the number, or try another card" not "Error 4022". On submit-with-errors: focus to the first error, summary announced for screen readers. `microcopy-writing` owns the sentences; this skill owns placement and timing.

5. **Configure the input layer per field — the cheap wins nobody does:** correct `type`/`inputmode` (the right mobile keyboard is a completion lever), `autocomplete` attributes (`cc-number`, `postal-code` — browser autofill completes half the form free; breaking it with clever custom inputs is self-sabotage), no format policing (accept spaces in card numbers — normalize server-side; formatting is your job, not the user's), paste never blocked (password managers exist; blocking paste on confirm fields is security theater with a UX bill).

6. **Multi-step when length demands it, with the honest trade:** split at natural sections, named progress, state persisted across steps AND accidental navigation (the back-button that eats twenty minutes is a rage-quit generator), completed steps revisitable. But steps add ceremony — a 6-field form as a 3-step wizard is theater; the split earns its place around 8+ fields or distinct phases.

7. **Instrument per-field and let the data drive iteration:** time-in-field, correction count, abandonment-at-field, error-rate-per-field. The error-rate table is the maintenance backlog: a field where 20% of users err is a design defect, not a user defect — fix the field, watch the metric (`metric-definition` hygiene on the completion rate's denominator).

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "Marketing needs the 'how did you hear about us' field" | Every field is measurable abandonment; the nice-for-marketing tax is paid in conversions. Collect it post-conversion or let marketing defend the cost with the funnel numbers. |
| "Placeholder labels look cleaner — modern design" | They vanish on focus, fail mid-fill recall, break autofill verification, and fail screen readers — four defects for one aesthetic. The most common form bug, and the cheapest fix. |
| "Validate as they type — instant feedback is good UX" | 'Invalid email' at the second character trains users that the form is hostile before any mistake exists. Blur-then-validate, instant-positive on success — the asymmetry IS the design. |
| "Rejecting badly-formatted input teaches the right format" | Rejecting `4242 4242 4242 4242` for its spaces punishes readability the server normalizes in one line. Format tolerance is a conversion lever wearing a shrug. |
| "The custom card input looks better than the native one" | And breaks autofill — whose users convert at multiples of manual typists. Aesthetics that tax the highest-converting cohort need the funnel math shown first. |
| "The form underperforms — time for a redesign" | The field-level data almost always convicts one or two specific fields (the example: two). Whole-form redesigns risk the fields that worked; instrument first, operate second. |

## Red Flags

- Fields nobody can name a consumer for.
- Placeholder-as-label anywhere.
- Red errors appearing mid-typing.
- Custom inputs with autofill broken; paste blocked on any field.
- The submit that visibly does nothing (double-submit tickets in support).
- Redesign proposals with no per-field funnel data attached.

## Verification

- [ ] Field audit done: each survivor's consumer named; deletions counted — list attached.
- [ ] Labels above fields; single column; minority marked — screenshots.
- [ ] Validation timing follows blur/positive-instant rules — behavior demonstrated.
- [ ] Errors at-field with icon+text+color and focus management — a11y check passed.
- [ ] Input layer configured: type/inputmode/autocomplete per field; paste allowed — attribute audit attached.
- [ ] Per-field instrumentation live — dashboard linked; completion metric's denominator defined.

## Example

Checkout form, 61% completion, redesign requested. Instrumentation first (step 7 as diagnosis): abandonment concentrated at two fields — VAT number (18% of EU users abandoned there; required for all, needed only by business buyers) and card number (high correction count; the field rejected spaces). Fixes, narrow: VAT behind a "buying as a business?" toggle (field deleted for 80% of users — step 1 by other means); card input accepting/auto-formatting spaces with `inputmode=numeric` + `autocomplete=cc-number` (autofill had been broken by a custom input — restored); validation moved from keystroke to blur with positive checkmarks; error copy rewritten at-field. No layout redesign at all. Completion 61% → 74% in three weeks, the VAT toggle alone worth ~8 points. The proposed full redesign was never needed — the two convicted fields were the whole story.

## Related skills

- `input-validation-boundaries` — the server-side rules these interactions surface.
- `microcopy-writing` — the words in labels, hints, and errors.
- `accessibility-audit` — announcement, focus, and labeling requirements.
- `ab-test-analysis` — verifying the redesign's lift honestly.
