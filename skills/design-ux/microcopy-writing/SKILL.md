---
name: microcopy-writing
description: >
  Use when writing the words inside a UI — buttons, errors, confirmations,
  empty states, tooltips — clarity over cleverness, at the moment of user
  need. Triggers: "button label", "error message wording", "what should
  this say", "confirmation dialog text", "UX writing", "tooltip copy".
  NOT for long-form content (see technical-blog-post, docs skills) or the
  visual/placement design of the states themselves.
---

# Microcopy Writing

## Overview

Interface words are read at a glance, mid-task, sometimes mid-crisis: buttons say what they do, errors say what happened and how to fix it, and one word per concept holds everywhere. Playfulness has a narrow license, revoked wherever the user is stressed.

## When to Use

- Writing or reviewing interface text: labels, buttons, errors, confirmations, hints, notifications.
- A support-ticket or analytics pattern traces to users misreading the UI.

**When NOT to use:**
- Long-form prose — `technical-blog-post`, docs skills.
- The states' placement/design — `form-design`, `empty-loading-error-states` (this skill supplies their sentences).

## Prerequisites

- The screen's context: what the user was doing, what they know at this moment, what happens after each choice — copy written without the flow open is caption-writing.

## The Workflow

1. **Buttons say what they DO, specifically:** verb + object naming the outcome — "Delete 3 invoices", "Send test email" — never bare "OK/Yes/Submit" on anything consequential. Paired buttons distinguishable without the surrounding prose: "Delete draft / Keep editing", not "Yes / No" (the dialog asking "Don't save?" with Yes/No has generated a decade of lost work).

2. **Errors: what happened + how to fix it, in the user's language:** the two-part contract, no codes-as-message, no blame framing ("The card was declined — check the number or try another card", not "Error 4022" / "You entered an incorrect value"). Reference IDs *available* for support without being the message. Never promise falsely ("Try again later" only if later will differ).

3. **Confirmations only for the irreversible, and then with real content:** destructive confirms state what's destroyed and its scope ("Delete 'Q3 Report' and its 14 responses? This can't be undone."), danger action visually distinct and NOT default-focused. Everything reversible skips the confirm and offers undo (the undo toast beats the confirm on safety AND speed — confirms train click-through; undo forgives). Confirm frequency is a design smell meter.

4. **Cut to the load-bearing words:** front-load the point ("Payment failed" before the why), one idea per string, no throat-clearing, courtesy trimmed on high-frequency surfaces (the 40th "Successfully saved your changes!" earns "Saved"). `natural-prose-editing`'s cuts at 6-word scale — without compressing past clarity ("Del. inv.?" saved characters and lost the meaning).

5. **Calibrate tone to the user's moment, not the brand's mood:** playfulness has a narrow license (success, empty states, onboarding — lightly); revoked at errors, payments, deletion, security, and anywhere the user arrives stressed ("Whoopsie! 🙈" on a failed payout reads as mockery). A one-line tone rule ("plain always; warm where calm") outperforms a brand-voice essay.

6. **Build and enforce the term sheet:** one word per concept, everywhere — the object is a "project" or a "workspace", never both; "remove" (reversible) vs "delete" (permanent) with the distinction USED consistently (terminology carrying semantics is free UX). The sheet lives in the design system; `ui-heuristic-review`'s consistency findings feed it; localization multiplies every violation.

7. **Test copy with the cheapest honest methods:** the cloze read-aloud (someone unfamiliar says what they think each control does — mismatches are findings), support-ticket mining (tickets quoting UI text back confusedly = that string's indictment), and A/B where volume allows (button copy is the classic cheap high-power test). Strings are shippable increments, not launch-day paint.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "Yes/No buttons are universal — everyone understands them" | Whose question? The 'Cancel (subscription) / Cancel (dialog)' collision ships everywhere, and the example's Yes/No dialog killed carts AND subscriptions. Name the object in the label. |
| "Users can figure out what Error 4022 means from context" | The schema is talking to the user, and the user is filing a ticket quoting it. Translate at the boundary: what happened + what to do, always. |
| "Add confirms everywhere — safety first" | Confirmation spam trains reflexive click-through, which defeats the ONE confirm that mattered. Undo for the reversible; confirms only where undo can't exist. |
| "Our brand voice is playful — keep it consistent everywhere" | Charm decays with repetition and detonates under stress: the pun in the payment failure reads as mockery to the person it just happened to. Tone follows the user's moment, not the brand deck. |
| "'Archive' here, 'hide' there — close enough" | Users construct a false mental model of two features, and localization doubles the damage. One word per concept is free UX; synonyms are an invisible tax. |
| "Copy tweaks can wait for the redesign" | Strings ship independently at near-zero cost — the example's four-string fix cut a 30-ticket/month cluster. Waiting bundles the cheapest fix in the building with the most expensive one. |

## Red Flags

- "OK / Cancel" on destructive dialogs.
- Raw enums or status codes user-visible.
- The same operation named differently on two screens.
- Confirm dialogs on reversible actions.
- Jokes inside error or payment surfaces.
- Support tickets quoting UI strings in confusion — unmined.

## Verification

- [ ] Every button in the diff names its outcome (verb + object); pairs distinguishable label-only — reviewed.
- [ ] Errors follow the two-part contract; zero raw codes as messages — spot-check.
- [ ] Confirms audited: each one either guards irreversibility or converts to undo — list attached.
- [ ] Term sheet consulted; new strings consistent — sheet linked, additions noted.
- [ ] Tone check: no playfulness in stress surfaces — reviewed against the tone rule.
- [ ] Cloze read-aloud (or ticket-mining evidence) done for the changed surfaces — findings noted.

## Example

Support mining flagged a cluster: ~30 tickets/month quoting the dialog "Are you sure you want to cancel? [Yes] [No]" — reached from BOTH the subscription page (canceling the plan) and mid-checkout (abandoning the form). Users clicked Yes meaning "yes, keep going with checkout" and killed their carts; two enterprise users canceled *subscriptions* believing they were dismissing a dialog. Rewrite: checkout version — "Leave checkout? Your cart is saved for 24 hours. [Leave checkout] [Keep shopping]"; subscription version — "Cancel your Pro subscription? You'll keep access until Aug 1. [Cancel subscription] [Keep my plan]", danger-styled, non-default-focus. Term sheet gained the entry: "cancel = subscriptions only; never for dialog dismissal." Tickets in the cluster: ~30/month → 2/month; the accidental-subscription-cancel became structurally impossible to do unknowingly. Total engineering cost: four strings and a focus attribute.

## Related skills

- `form-design` — placement and timing of these words in forms.
- `empty-loading-error-states` — the states these sentences inhabit.
- `natural-prose-editing` — the same clarity surgery at paragraph scale.
- `ab-test-analysis` — testing the strings that carry conversion weight.
