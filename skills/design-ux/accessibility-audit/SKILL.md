---
name: accessibility-audit
description: >
  Use when checking a UI for accessibility — keyboard operability,
  contrast, semantic structure, screen-reader sanity, focus management —
  as a WCAG-informed practical pass, not a checkbox exercise. Triggers:
  "accessibility audit", "is this accessible", "WCAG check", "keyboard
  navigation broken", "screen reader support", "a11y review". NOT a
  legal-conformance certification (VPAT/audit-for-lawsuit needs
  specialists) — this pass finds and fixes the real barriers.
---

# Accessibility Audit

## Overview

The scanner catches ~30–40%; the barriers live in what it can't see — keyboard traps, focus management on dynamic changes, silent async updates, reading order. The audit is the mouse unplugged, the screen reader on, and fixes landed in shared components where one repair heals every instance.

## When to Use

- Auditing a product/feature before release or after complaints.
- Setting up a team's recurring a11y checks.

**When NOT to use:**
- Formal conformance claims — specialists; this pass is most of the substance underneath one.

## Prerequisites

- The running UI; a screen reader at basic operating level (NVDA/VoiceOver — 20 minutes of practice suffices for auditing); an axe-style devtools extension.
- The 2–4 core user tasks — flows over isolated screens.

## The Workflow

1. **Run the automated scan first, knowing its limits:** axe/Lighthouse catches contrast, missing alt, ARIA misuse, label associations in minutes — fix the mechanical findings, and treat "0 automated errors" as the *starting line*. Everything after is what the scanner can't see.

2. **Unplug the mouse: complete every task by keyboard alone.** Hunting: unreachable controls (click-handlers on divs — the classic), no visible focus indicator, illogical focus order, keyboard traps, and the killer — **focus management on dynamic changes:** modal opens (focus in? trapped? returns on close?), item deleted (focus somewhere sane?), SPA route change (focus + announcement, or the reader user doesn't know the page changed).

3. **Then the same tasks with the screen reader,** listening for: every control announcing name + role + state; alt texts informative-or-empty-by-choice; errors and async updates announcing (live regions — the visually-obvious spinner that's silence to a listener); coherent reading order; form errors associated to fields (`aria-describedby`).

4. **Verify the semantic skeleton in the DOM:** one `<h1>`, unskipped heading levels (reader users navigate by heading list — a page of styled divs has no skeleton), landmarks, lists as lists, buttons as `<button>` (the div-with-onclick fails keyboard, role, AND state at once — native elements are three fixes in one). Rule with teeth: **no ARIA beats bad ARIA** — `role="button"` without keydown handling is a lie the screen reader repeats.

5. **Check the visual dimension beyond the scanner:** contrast on states the scan missed (hover, focus, disabled, placeholder), information carried by color alone (add icons/labels), zoom at 200% and 400% (reflow or clipped? — low-vision users' primary tool), `prefers-reduced-motion` respected, touch targets ≥ ~44px.

6. **Rate by user impact, fix by root cause:** blocker (task impossible for a group) / major / minor; cluster to root causes — forty missing labels is ONE finding ("no label discipline in the form library") with one systemic fix. Fix in the shared component: the design system's Button fixed once fixes every button (`design-system-tokens` is the leverage point).

7. **Institutionalize so the audit isn't annual archaeology:** axe in CI (component tests/Storybook — regressions caught at PR time), the keyboard-walk in feature QA's definition-of-done, a11y acceptance criteria in component specs (`wireframe-to-spec`). One audit fixes a snapshot; the pipeline changes fix the trajectory — retrofit costs 5–10× the build-it-in price.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "The scanner's clean — we're accessible" | The scanner sees 30–40%; keyboard traps, focus management, and silent async live in the other 60 — which is where the actual blockers are. Scanner-clean is the starting line. |
| "Add aria-labels everywhere — that's the fix" | ARIA pixie dust creates lies the screen reader repeats: roles contradicting behavior, hidden-but-tabbable ghosts. Native semantics first; ARIA only for genuine gaps, tested with a reader. |
| "The focus outline is ugly — remove it, designers insist" | outline:none unremediated navigates keyboard users blind. Style it to the brand; deleting it is choosing aesthetics over an entire input modality. |
| "Alt text is done — every image has one" | 'image123.png' and 'image of chart' are worse than none. Informative, equivalent, or empty-by-choice; the chart's alt owes the DATA. |
| "We'll do the a11y pass at the end of the quarter" | Retrofit runs 5–10× the build-in price, and the end-of-quarter pass loses to roadmap pressure every time. CI + definition-of-done (step 7) is the only version that survives. |
| "Nobody's complained, so it's not blocking anyone" | Blocked users leave silently; complaints are the tip that clears the iceberg. The keyboard-walk takes an hour and measures the actual barrier, not the complaint rate. |

## Red Flags

- Any interactive div-with-onclick in new code.
- Global `outline: none` with no replacement.
- Modals that don't trap or return focus.
- Async results with no live-region announcements.
- Heading levels skipping (h1 → h4).
- A11y items perpetually in "polish" backlogs.
- Zero a11y checks in CI.

## Verification

- [ ] Automated scan clean or dispositioned — report linked.
- [ ] All core tasks completable keyboard-only — walk recorded, defects listed.
- [ ] Screen-reader pass done: names/roles/states, announcements, reading order — findings listed.
- [ ] Semantic skeleton verified (headings, landmarks, native elements) — DOM check noted.
- [ ] Zoom/contrast/motion/touch checks done at 200% and 400% — results noted.
- [ ] Fixes landed at component-library level where clustered — PRs linked.
- [ ] CI + DoD wiring live — job link; regressions caught count tracked.

## Example

Audit target: signup + first-project flow, prompted by a customer's a11y procurement questionnaire. Scan: 23 automated findings (18 mechanical — fixed same day). Keyboard walk: plan-selection cards were divs-with-onclick — unreachable entirely (blocker); the email-verification modal trapped focus correctly but *returned focus to <body>* on close, stranding the user at page top (major). Screen-reader pass: the async "checking availability..." on username was visual-only — listeners got silence then an unexplained disabled button (major, live-region fix); the progress stepper announced "list, 4 items" with no current-step state (`aria-current`, minor). Root-cause clustering: the div-buttons all came from one Card component — fixed once in the library (native button + focus style), 14 instances healed. Institutionalized: axe on Storybook in CI (caught 3 regressions in the first quarter), keyboard-walk added to QA. The questionnaire got honest answers, and the deal that prompted the audit closed.

## Related skills

- `ui-heuristic-review` — the sibling pass for general usability.
- `design-system-tokens` — the component-level leverage for systemic fixes.
- `form-design` — where a11y and usability findings concentrate.
- `wireframe-to-spec` — specifying announcements/focus before build.
