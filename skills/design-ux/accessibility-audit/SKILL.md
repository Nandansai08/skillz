---
name: accessibility-audit
description: >
  Use when checking a UI for accessibility — keyboard operability, contrast,
  semantic structure, screen-reader sanity, focus management — as a WCAG-
  informed practical pass, not a checkbox exercise. Triggers: "accessibility
  audit", "is this accessible", "WCAG check", "keyboard navigation broken",
  "screen reader support", "a11y review".
---

# Accessibility Audit

## When to use this skill
- Auditing a product/feature for accessibility before release or after complaints.
- Setting up the recurring a11y checks for a team.
- NOT a legal-compliance certification — formal conformance claims (VPAT, audit-for-lawsuit) need specialists; this pass finds and fixes the real barriers, which is also most of conformance.

## Prerequisites
- The running UI; a screen reader you can operate at basic level (NVDA on Windows, VoiceOver on Mac — 20 minutes of practice suffices for auditing); browser devtools with an axe-style extension.
- The 2–4 core user tasks — like `ui-heuristic-review`, task flows over isolated screens.

## Workflow

1. **Run the automated scan first, knowing its limits:** axe/Lighthouse catches ~30–40% of issues (contrast, missing alt, ARIA misuse, label associations) in minutes — run it, fix the mechanical findings, and treat "0 automated errors" as the *starting line*, not the certificate. Everything after this step is what the scanner can't see.

2. **Unplug the mouse: complete every task by keyboard alone.** Tab/Shift-Tab/Enter/Space/Esc/arrows. Hunting for: unreachable controls (click-handlers on divs — the classic), no visible focus indicator (`:focus-visible` styled, contrast-sufficient), focus order that jumps illogically (DOM order vs visual order fights), keyboard traps (can't Esc the datepicker), and the killer: **focus management on dynamic changes** — modal opens (focus moves in? trapped? returns on close?), item deleted (focus goes somewhere sane, not to <body>?), route change in a SPA (focus + announcement, or the screen reader user doesn't know the page changed?).

3. **Then the same tasks with the screen reader**, listening for: does every control announce name + role + state ("Save, button" / "Notifications, toggle, off")? Are images' alt texts informative-or-empty-by-choice (`alt=""` for decorative — an alt of "image123.png" is worse than none)? Do errors and async updates announce (live regions: `aria-live="polite"` on toasts, `role="alert"` on failures — the visually-obvious spinner that's silence to a listener)? Is the reading order coherent? Forms: does each error announce, associated to its field (`aria-describedby`)?

4. **Verify the semantic skeleton in the DOM:** one `<h1>`, heading levels unskipped (screen-reader users navigate by heading list — a page of styled `<div>`s has no skeleton), landmarks (`<main>`, `<nav>`, `<header>`), lists as lists, buttons as `<button>` and links as `<a>` (the div-with-onclick fails keyboard, role, AND state announcement simultaneously — native elements are three fixes in one). Rule with teeth: **no ARIA beats bad ARIA** — `role="button"` without keydown handling is a lie the screen reader repeats to its user.

5. **Check the visual dimension beyond the scanner:** contrast on states the scanner missed (hover, focus, disabled, placeholder, text-on-images); information carried by color alone (the red/green status dots — add icons/labels); zoom to 200% and 400% (content reflows? or clipped/overlapped? — this is low-vision users' primary tool); motion (autoplaying animation respects `prefers-reduced-motion`?); touch targets ≥ ~44px on mobile.

6. **Rate findings by user impact, fix by root cause:** blocker (task impossible for a group: keyboard trap, unlabeled form) / major (possible but punishing) / minor. Cluster to root causes like `ui-heuristic-review` step 6 — forty missing labels is one finding ("no label discipline in the form library") with one systemic fix. Fix in the shared component, not per-instance: the design system's Button fixed once fixes every button (`design-system-tokens` is the leverage point).

7. **Institutionalize so the audit isn't annual archaeology:** automated checks in CI (axe on component tests/Storybook — catches regressions at PR time), the keyboard-walk as part of feature QA definition-of-done, a11y acceptance criteria in component specs (`wireframe-to-spec` step includes states + announcements), and the finding categories fed back into component-library docs. One audit fixes a snapshot; the pipeline changes fix the trajectory.

## Common pitfalls
- Scanner-clean declared done — the 60% the scanner can't test (focus management, announcement, reading order, cognitive load) is where the actual blockers live (steps 2–3 are the audit).
- ARIA sprinkled as pixie dust: `aria-label` on everything, roles contradicting behavior, `aria-hidden` on focusable content (invisible to reader, still tabbable — a ghost stop). Native semantics first; ARIA only to fill genuine gaps, tested with a reader.
- Focus outline removed globally for aesthetics (`outline: none`) and never replaced — keyboard users navigate blind. Style it, never just delete it.
- Auditing only static screens: the modal, toast, infinite-scroll, and route-change behaviors — the dynamic focus/announcement layer — is where modern SPAs fail hardest (step 2's killer list).
- Alt-text theater: every image dutifully alt-texted with its filename or "image of chart" — the chart's *data* is what the alt owed. Informative, equivalent, or empty-by-choice.
- Treating a11y as a sprint-end pass forever: retrofit costs 5–10× the build-it-in price; step 7's CI + DoD wiring is the only version that survives roadmap pressure.

## Example
Audit target: signup + first-project flow, prompted by a customer's a11y procurement questionnaire. Scan: 23 automated findings (18 contrast/label mechanical — fixed same day). Keyboard walk: plan-selection cards were divs-with-onclick — unreachable entirely (blocker); modal for email verification trapped focus correctly but *returned focus to <body>* on close, stranding the user at page top (major). Screen-reader pass: the async "checking availability..." on username was visual-only — listeners got silence then an unexplained disabled button (major, live-region fix); progress stepper announced "list, 4 items" with no current-step state (`aria-current` fix, minor-turned-easy). Root-cause clustering: the div-buttons all came from one Card component — fixed once in the library (native button + focus style), 14 instances healed. Institutionalized: axe on Storybook in CI (caught 3 regressions in the first quarter), keyboard-walk added to the QA checklist. The procurement questionnaire got honest answers, and the sales team got a one-pager of what was fixed — which closed the deal that prompted the audit.

## Related skills
- `ui-heuristic-review` — the sibling pass for general usability.
- `design-system-tokens` — the component-level leverage for systemic fixes.
- `form-design` — where a11y and usability findings concentrate.
- `security-headers-config` — a neighboring "floor checklist" discipline, different domain.
