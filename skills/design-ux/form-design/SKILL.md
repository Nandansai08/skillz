---
name: form-design
description: >
  Use when designing or fixing forms — field ordering, validation timing,
  error copy placement, mobile input types, reducing abandonment.
  Triggers: "design this form", "form abandonment is high", "validation UX",
  "checkout form", "signup form review", "too many form errors".
---

# Form Design

## When to use this skill
- Building any form beyond trivial (signup, checkout, settings, multi-step applications).
- A form's completion/abandonment metrics are bad and need diagnosis.
- NOT for the backend validation rules themselves — `input-validation-boundaries` owns where checks live; this skill owns how humans experience them (the two must agree on the rules).

## Prerequisites
- The list of fields with each one's justification — and the authority to challenge it (the best form-design tool is deletion).

## Workflow

1. **Delete fields before designing them:** for each field — is it needed NOW (vs collectable later, post-conversion)? Derivable (city from postal code, card type from number)? Merely nice-for-marketing (the "how did you hear about us" tax)? Every removed field is measurable abandonment recovered; the field that survives must name its consumer. Optional fields either justify their presence or leave; if kept, mark *required* fields only when most are optional, and vice versa — mark the minority.

2. **Order and group by the user's mental flow:** logical sections matching how users think about the task (who you are → where to ship → how to pay), easy-before-invasive (asking email before card number; asking phone last, with the "why we need this" line), single column (multi-column forms cause skipped fields and broken tab order — the research here is unusually unanimous), labels *above* fields (not placeholder-as-label: it vanishes on focus, fails recall mid-fill, and fails `accessibility-audit` step 3's announcement check).

3. **Time validation by the "reward early, punish late" rule:** validate a field on blur-after-input (not on every keystroke while they're still typing — premature red is hostile), EXCEPT flip to instant-positive once a field goes valid (the checkmark appearing as they finish typing the email). Never validate-on-keystroke for incomplete input ("invalid email" while they've typed `jo`); never save all validation for submit (the scroll-back-up error hunt is the abandonment machine). Async checks (username availability) show their pending state inline and announce it (`aria-live` — the a11y audit's silent-spinner finding).

4. **Write errors as fix-instructions, placed at the field:** what's wrong + how to fix, in human words, next to the field it concerns, with the field's state visibly marked (color + icon + text — never color alone). "Card declined — check the number, or try another card" not "Transaction error 4022". On submit-with-errors: focus moves to the first error, and a summary announces for screen readers. `microcopy-writing` owns the sentences; this skill owns their placement and timing.

5. **Configure the input layer per field — the cheap wins nobody does:** correct `type`/`inputmode` (email/tel/numeric — the right mobile keyboard is a completion-rate lever), `autocomplete` attributes (`given-name`, `postal-code`, `cc-number` — browser autofill completes half the form for free; breaking it with clever custom inputs is self-sabotage), no premature format policing (accept spaces in card numbers, dashes in phones — normalize server-side per `data-cleaning-pipeline` instincts; formatting is your job, not the user's), sensible masks only where they genuinely help (dates), paste never blocked (password managers exist; blocking paste on the confirm field is security theater with a UX bill).

6. **Multi-step when length demands it, with the honest trade:** split at natural section boundaries, show progress (steps named, not just "3 of 7"), persist state across steps AND across accidental navigation (the back-button that eats twenty minutes of form data is a rage-quit generator), allow revisiting completed steps. But: steps add friction ceremony — a 6-field form as a 3-step wizard is theater; the split earns its place around ~8+ fields or genuinely distinct phases.

7. **Instrument per-field and let the data drive iteration:** field-level analytics (time-in-field, correction count, abandonment-at-field, error-rate-per-field) turn "the form underperforms" into "the VAT field is where 30% die." The error-rate-per-field table is the maintenance backlog: a field where 20% of users err is a design defect, not a user defect — fix the field (better label, example text, looser format acceptance), and watch the metric (`metric-definition` hygiene applies: completion rate's denominator = started, defined precisely).

## Common pitfalls
- Placeholder-as-label: vanishes on focus, invisible to recall, fails autofill verification and accessibility simultaneously — the single most common form defect, and the cheapest to fix.
- Keystroke-eager validation screaming "invalid" at half-typed input — trains users that the form is hostile before they've made any mistake.
- Format tyranny: rejecting `4242 4242 4242 4242` for its spaces, phone numbers for their dashes — punishing users for readability the server could normalize in one line.
- The silent submit failure: button clicked, nothing visibly happens (error rendered off-viewport, or swallowed) — users click again (double-submit — `api-design-rest` step 5's idempotency key is the backend twin) or leave.
- Blocked autofill/paste from custom-component vanity — measured autofill users convert at multiples of manual typists; breaking their path is burning conversion for aesthetics.
- Redesigning the form on taste when field-level data exists — the instrumentation (step 7) almost always convicts one or two specific fields, and the whole-form redesign risks the fields that worked.

## Example
Checkout form, 61% completion, redesign requested. Instrumentation first (step 7 run as diagnosis): abandonment concentrated at two fields — VAT number (18% of EU users abandoned there; it was required for all, needed only for business buyers) and card number (high correction count; the field rejected spaces). Fixes, narrow: VAT behind a "buying as a business?" toggle (field deleted for 80% of users — step 1 by other means); card input accepting/auto-formatting spaces with `inputmode=numeric` + `autocomplete=cc-number` (autofill had been broken by a custom input — restored); validation timing moved from keystroke to blur with positive checkmarks; error copy rewritten at-field ("check the number — this one's a digit short"). No layout redesign at all. Completion 61% → 74% in three weeks, the VAT toggle alone worth ~8 points. The proposed full visual redesign, mercifully, was never needed — the two convicted fields had been the whole story.

## Related skills
- `input-validation-boundaries` — the server-side rules these interactions surface.
- `microcopy-writing` — the words in labels, hints, and errors.
- `accessibility-audit` — announcement, focus, and labeling requirements.
- `ab-test-analysis` — verifying the redesign's lift honestly.
