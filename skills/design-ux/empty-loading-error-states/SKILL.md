---
name: empty-loading-error-states
description: >
  Use when designing the non-happy-path states of a UI — empty, loading,
  error, partial, and offline states, designed before the ideal state
  gets all the attention. Triggers: "empty state design", "loading
  states", "what happens when there's no data", "error state UX",
  "skeleton screens", "the demo looked great but real accounts look
  broken". NOT for the error-handling logic itself (see
  error-handling-strategy) — this designs what the human sees.
---

# Empty, Loading & Error States

## Overview

Every screen backed by dynamic data has five states per data source, and the ones nobody enumerated ship as blank divs. The matrix written into the spec, empties designed by cause, loading matched to duration, and errors that always carry an exit — that's the difference between the demo and the product.

## When to Use

- Designing/reviewing any screen backed by dynamic data (which is every screen worth designing).
- The product demos beautifully with seeded data and looks abandoned on real new accounts.

**When NOT to use:**
- The failure logic itself — `error-handling-strategy`; this skill designs its visible surface.

## Prerequisites

- The screen's data dependencies enumerated: every query/call it renders from — each one is a state-matrix row.

## The Workflow

1. **Enumerate the full state matrix before designing the ideal state:** per data dependency: ideal / empty / loading / error / partial. A screen with three sources has combinations ("list loaded, counts failed"). The matrix goes in the spec (`wireframe-to-spec` demands it) — the states nobody enumerated are the states that ship as blank divs.

2. **Design empty states by their cause — three different designs, not one illustration:**
   - **First-use empty:** onboarding real estate, not absence — what will appear here, why it's valuable, the ONE action that fills it ("No invoices yet — create your first, or import from Stripe").
   - **Cleared empty:** quiet confirmation — inbox-zero tone, not salesmanship.
   - **No-results empty:** restate what was searched, offer the exits (clear filters) — and never look like first-use ("It appears you have no invoices" atop 4,000 filtered-out invoices erodes trust in the data itself).

3. **Match loading treatment to expected duration, and keep layout stable:** <300ms — show nothing (a flashed spinner reads as jank); 300ms–2s — skeletons shaped like the coming content; >2s — progress with communication. The iron rule across all: **no layout shift** — content lands in reserved space (the mis-click on the button that moved is a state-design failure).

4. **Design errors by recoverability, always with an exit:**
   - **Retryable** (network, timeout): plain statement + retry (idempotency backing it — `api-design-rest` step 5).
   - **User-fixable:** what to fix and where (`form-design` step 4's at-field discipline).
   - **Dead-end:** honest apology, error reference ID, and a path OUT — never a modal-with-no-close. Preserve the user's work in all cases (draft, scroll position, filters) — the error that eats input converts a bug into a betrayal.
   Tone: plain, unblaming ("Whoopsie!" on a payment failure reads as mockery — `microcopy-writing` calibrates).

5. **Handle partial states as first-class, not as tiny errors:** three of four widgets loaded — render the three, degrade the fourth in-place with its own retry, rather than failing the page; stale-while-revalidating data gets a freshness indicator where staleness matters. Partials are the *most common* degraded state in production and the least designed — block, degrade, or hide, decided per widget (`error-handling-strategy` step 5's loud-fallback rule, rendered).

6. **Wire the states to reality with test data and forced conditions:** a state-forcing mechanism in dev/Storybook (every component demo-able in all five states — where `accessibility-audit`'s announcements get checked too), network throttling and failure injection in QA, and the demo-account trap inverted — a "day-one account" fixture every new screen gets viewed with before ship.

7. **Audit shipped surfaces against the matrix quarterly-ish:** the live product's actual empty/error surfaces walked (new account created, network killed mid-session, one API blackholed) — regressions here are invisible in normal dogfooding because employees' accounts are never empty and the office network never flakes.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "Design the main experience first — states are polish" | The paying customer's first hour IS the empty state, and their bad-wifi commute IS the error state. The 'polish' is most users' actual first impression. |
| "One nice empty illustration covers all the empty cases" | The no-results case telling a user with 4,000 filtered invoices 'nothing here yet!' erodes trust in the data itself. Three causes, three designs (step 2). |
| "A spinner handles loading — universal solution" | The 80ms fetch flashing a spinner is jank; the 4s load behind a bare spinner is abandonment; and the page-level spinner blocking arrived content is both. Duration-matched treatment (step 3). |
| "Errors are rare — a generic 'something went wrong' suffices" | Without retry, exit, or work-preservation, the rare error converts a bug into a betrayal — and partial failures aren't rare at all (step 5). |
| "We test with the seeded demo account — it covers the UI" | Seeded accounts are never empty and office networks never flake — which is exactly HOW empty and error states ship broken. The day-one fixture and the blackhole pass (step 6) exist for this. |
| "The states are coded — no need for a forcing mechanism" | Unreachable-in-dev states rot invisibly; the error state nobody's seen since it was coded has been broken for two quarters. Demo-able states are testable states. |

## Red Flags

- Mockups/specs showing ideal-state only.
- The same illustration on first-use and no-results empties.
- Spinners flashing on sub-second fetches; page-level spinners over partial content.
- Layout visibly shifting as content arrives.
- Error states with no retry, no exit, or lost user input.
- One failed widget blanking a whole dashboard.
- No Storybook/forcing states for non-happy paths.

## Verification

- [ ] State matrix complete in the spec: every data dependency × five states dispositioned — linked.
- [ ] Three empty causes designed distinctly where applicable — designs shown.
- [ ] Loading treatments duration-matched; zero layout shift demonstrated (CLS or recording).
- [ ] Every error state has its recoverability class, an exit, and work-preservation — walkthrough.
- [ ] Partial-state decisions per widget (block/degrade/hide) — documented.
- [ ] Forcing mechanism live for all states — Storybook links; day-one-account review done before ship.

## Example

Dashboard redesign review, caught pre-ship by the matrix exercise: four widgets, spec covered ideal-only. Matrix pass produced 20 cells; 11 undesigned. Decisions made explicitly: page renders on ANY widget success (partials in-place-degraded with per-widget retry — previously one failed call blanked the page); skeletons shaped per-widget with reserved heights (the old spinner had caused the layout-shift mis-click support kept hearing about); first-use empties designed as setup prompts per widget ("Connect Stripe to see revenue") — which turned the new-account dashboard from a void into an onboarding checklist, lifting week-1 connection rate +11%; stale indicator on the revenue widget only. Storybook forcing states wired; QA added the blackhole-one-API pass. The quarterly audit six months later found one regression (a new widget shipped matrix-less, blank on empty) — one, versus the pre-discipline baseline of "all of them."

## Related skills

- `error-handling-strategy` — the code-side truth these states render.
- `form-design` — the form-specific error state discipline.
- `microcopy-writing` — the sentences inside every one of these states.
- `wireframe-to-spec` — where the state matrix becomes a deliverable.
