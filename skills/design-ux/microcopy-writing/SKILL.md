---
name: microcopy-writing
description: >
  Use when writing the words inside a UI — buttons, errors, confirmations,
  empty states, tooltips — clarity over cleverness, at the moment of user
  need. Triggers: "button label", "error message wording", "what should
  this say", "confirmation dialog text", "UX writing", "tooltip copy".
---

# Microcopy Writing

## When to use this skill
- Writing or reviewing interface text: labels, buttons, errors, confirmations, hints, notifications.
- A support-ticket or analytics pattern traces to users misreading the UI.
- NOT for long-form content (`technical-blog-post`, docs) or the visual/placement design of the states themselves (`form-design`, `empty-loading-error-states` — this skill supplies their sentences).

## Prerequisites
- The screen's context: what the user was doing, what they know at this moment, what happens after each choice — copy written without the flow open is caption-writing, not interface writing.

## Workflow

1. **Buttons say what they DO, specifically:** verb + object naming the outcome — "Delete 3 invoices", "Send test email", "Save and close" — never bare "OK/Yes/Submit/Continue" on anything consequential (the button label is the last thing read before commitment; it should be answerable evidence of what's about to happen). Paired buttons must be distinguishable without the surrounding prose: "Delete draft / Keep editing", not "Yes / No" (whose question? — the dialog that asks "Don't save?" with Yes/No buttons has generated a decade of lost work).

2. **Errors: what happened + how to fix it, in the user's language:** the two-part contract, no codes-as-message, no blame framing ("The card was declined — check the number or try another card", not "Invalid input" / "Error 4022" / "You entered an incorrect value"). Keep the reference ID *available* for support without making it the message (`error-handling-strategy` step 4's typed errors carry the data; this skill renders them humane). Never promise falsely ("Try again later" only if later will actually differ).

3. **Confirmations only for the irreversible, and then with real friction-content:** destructive confirms state what's being destroyed and its scope ("Delete 'Q3 Report' and its 14 responses? This can't be undone."), with the danger action visually distinct and NOT the default-focused button. Everything reversible skips the confirm and offers undo instead (the undo toast beats the confirm dialog on both safety AND speed — confirms train click-through; undo forgives). Confirm-dialog frequency is a design smell meter: many confirms = the flow is scary instead of safe.

4. **Cut to the load-bearing words:** UI text is read at a glance mid-task — front-load the point ("Payment failed" before the why), one idea per string, no throat-clearing ("Please note that..."), no redundant courtesy padding on high-frequency surfaces (the 40th "Successfully saved your changes!" of the day earns its trim to "Saved"). `natural-prose-editing`'s cuts, at 6-word scale — with the same caution: don't compress past clarity; "Del. inv.?" saved characters and lost the meaning.

5. **Calibrate tone to the user's moment, not the brand's mood:** playfulness has a narrow license (success states, empty states, onboarding — lightly); it's revoked at errors, payments, deletion, security, and anything the user reaches while stressed ("Whoopsie! 🙈" on a failed payout reads as mockery — the more the moment matters to the user, the plainer the voice). A one-line tone rule in the term sheet ("plain always; warm where calm") outperforms a brand-voice essay.

6. **Build and enforce the term sheet:** one word per concept, everywhere — the object is a "project" or a "workspace", never both; the action is "remove" (reversible) vs "delete" (permanent) with the distinction USED consistently (terminology carrying semantics is free UX); the sheet lives in the design system next to the tokens (`design-system-tokens` for words), and `ui-heuristic-review`'s consistency findings feed it. Localization multiplies every violation — the term sheet is also the translators' contract.

7. **Test copy with the cheapest honest methods:** the cloze read-aloud (someone unfamiliar reads the screen and says what they think each control does — mismatches are findings), support-ticket mining (tickets quoting UI text back confusedly = that string's indictment), and A/B where volume allows (`ab-test-analysis` — button copy is the classic cheap high-power test). Copy is the most testable, most iterable layer of the interface; treat strings as shippable increments, not launch-day paint.

## Common pitfalls
- Ambiguous button pairs on destructive dialogs — the "Cancel (the subscription) / Cancel (this dialog)" collision, genuinely shipped everywhere. Name the object in the label.
- System-voice leakage: "Invalid entity mapping", "Session token expired", enum values surfacing raw — the schema talking to the user (`ui-heuristic-review` heuristic 2). Translate at the boundary, always.
- Cleverness where clarity was owed: the pun in the error, the mascot in the payment failure, the "Oopsie!" — charm decays with repetition and detonates under stress (step 5's license terms).
- Confirmation spam as safety theater: confirms on reversible actions train reflexive click-through, which then defeats the one confirm that mattered (step 3's undo-first rule).
- Inconsistent vocabulary as invisible tax: "archive" here, "hide" there, same operation — users construct a false mental model of two features. The term sheet is cheap; the confusion isn't.
- Writing strings in a spreadsheet without the screen: grammatical, polite, and wrong for the moment — context is the medium (prerequisites are not optional).

## Example
Support mining flagged a cluster: ~30 tickets/month quoting the dialog "Are you sure you want to cancel? [Yes] [No]" — reached from BOTH the subscription page (canceling the plan) and mid-checkout (abandoning the form). Users clicked Yes meaning "yes, keep going with checkout" and killed their carts; two enterprise users canceled *subscriptions* believing they were dismissing a dialog. Rewrite per the rules: checkout version — "Leave checkout? Your cart is saved for 24 hours. [Leave checkout] [Keep shopping]"; subscription version — "Cancel your Pro subscription? You'll keep access until Aug 1. [Cancel subscription] [Keep my plan]", danger-styled, non-default-focus. Term sheet gained the entry: "cancel = subscriptions only; never for dialog dismissal — use Close/Keep/Back." Tickets in the cluster: ~30/month → 2/month; the accidental-subscription-cancel became structurally impossible to do unknowingly. Total engineering cost: four strings and a focus attribute.

## Related skills
- `form-design` — placement and timing of these words in forms.
- `empty-loading-error-states` — the states these sentences inhabit.
- `natural-prose-editing` — the same clarity surgery at paragraph scale.
- `ab-test-analysis` — testing the strings that carry conversion weight.
