---
name: empty-loading-error-states
description: >
  Use when designing the non-happy-path states of a UI — empty, loading,
  error, partial, and offline states, designed before the ideal state gets
  all the attention. Triggers: "empty state design", "loading states",
  "what happens when there's no data", "error state UX", "skeleton screens",
  "the demo looked great but real accounts look broken".
---

# Empty, Loading & Error States

## When to use this skill
- Designing/reviewing any screen backed by dynamic data (which is every screen worth designing).
- The product demos beautifully with seeded data and looks abandoned on real new accounts.
- NOT for the error-handling logic itself — `error-handling-strategy` decides what fails and how; this skill designs what the human sees when it does.

## Prerequisites
- The screen's data dependencies enumerated: every query/call it renders from — each one is a state-matrix row.

## Workflow

1. **Enumerate the full state matrix before designing the ideal state:** per data dependency: ideal / empty / loading / error / partial. Five states per source, and a screen with three sources has combinations ("list loaded, but counts failed"). The discipline: the matrix is written down in the spec (`wireframe-to-spec` demands it), because the states nobody enumerated are the states that ship as blank divs.

2. **Design empty states by their cause — three different designs, not one illustration:**
   - **First-use empty** (new account): this is onboarding real estate, not absence — say what will appear here, why it's valuable, and the ONE action that fills it ("No invoices yet — create your first, or import from Stripe"), ideally with sample/demo content offerable.
   - **Cleared empty** (user emptied it): quiet confirmation, not celebration or salesmanship — inbox-zero tone.
   - **No-results empty** (filters/search excluded everything): restate what was searched, offer the exits (clear filters, adjust spelling) — and never look like an error or like first-use ("It appears you have no invoices" atop an account with 4,000 filtered-out invoices erodes trust in the data itself).

3. **Match loading treatment to expected duration, and keep layout stable:** <300ms — show nothing (a flashed spinner reads as jank); 300ms–2s — skeleton screens shaped like the coming content (they set expectations AND reserve space); >2s or genuinely long operations — progress with communication (what's happening, determinate bar where honest, background-it with notification where possible). The iron rule across all: **no layout shift** — content must land in reserved space, not shove the page (the mis-click caused by a late-loading banner is a state-design failure).

4. **Design errors by recoverability, always with an exit:**
   - **Retryable** (network, timeout): say it plainly + retry button (idempotency backing the retry — `api-design-rest` step 5) + auto-retry with backoff where sensible.
   - **User-fixable** (validation, permissions): what to fix and where — `form-design` step 4's at-the-field discipline.
   - **Dead-end** (server fault): honest apology, error reference ID for support, and a path OUT — never a modal-with-no-close, never a blank page. Preserve the user's work in all cases (their draft, their scroll position, their filters) — the error that eats the user's input converts a bug into a betrayal.
   Tone: plain, unblaming, no cutesy overload ("Whoopsie!" on a payment failure reads as mockery — `microcopy-writing` owns calibration).

5. **Handle partial states as first-class, not as tiny errors:** three of four widgets loaded — render the three, degrade the fourth in-place with its own retry ("Revenue chart unavailable — retry") rather than failing the page; stale-while-revalidating data gets a freshness indicator when the staleness matters (dashboards: "as of 14:02"). Partials are the *most common* degraded state in production and the least designed — decide per widget: block, degrade, or hide, explicitly (`error-handling-strategy` step 5's loud-fallback rule, rendered).

6. **Wire the states to reality with test data and forced conditions:** a state-forcing mechanism in dev/storybook (each component demo-able in all five states — this is where `accessibility-audit`'s announcements get checked too: loading and error changes must announce), network throttling and API-failure injection as part of QA, and the demo-account trap inverted — a "day-one account" fixture that every new screen gets viewed with before ship (the seeded-data-only review is HOW empty states ship broken).

7. **Audit shipped surfaces against the matrix quarterly-ish:** the live product's actual empty/error surfaces walked (new account created, network killed mid-session, a widget's API blackholed) — regressions here are invisible in normal dogfooding because employees' accounts are never empty and the office network never flakes. Findings cluster per `ui-heuristic-review` step 6 and fix in shared components for leverage.

## Common pitfalls
- Ideal-state-only design: mockups and demos with perfect seeded data; the paying customer's first hour is blank divs and unstyled error text. The matrix (step 1) in the spec is the structural fix.
- One generic empty illustration reused for all three empty causes — the no-results case telling a user with thousands of records "nothing here yet!" (step 2's trust erosion).
- Spinner maximalism: every 80ms fetch flashing a loader (jank), or one page-level spinner blocking content that already arrived (step 3 + step 5).
- Error states that strand: the retry-less failure, the modal with no escape, the error page with no navigation — every error needs an exit and an action (step 4).
- Layout shift as loading strategy: content shoving down as pieces arrive, the mis-click on the button that moved (step 3's iron rule).
- States built but unreachable in dev: no forcing mechanism, so nobody's seen the error state since it was coded — and it's been broken for two quarters (step 6).

## Example
Dashboard redesign review, caught pre-ship by the matrix exercise: four widgets, and the spec covered ideal-only. Matrix pass produced 20 cells; the review found 11 undesigned. Decisions made explicitly: page renders on ANY widget success (partials in-place-degraded with per-widget retry — previously one failed call blanked the page); skeletons shaped per-widget with reserved heights (the old spinner had caused the layout-shift mis-click support kept hearing about); first-use empties designed as setup prompts per widget ("Connect Stripe to see revenue") — which turned the new-account dashboard from a void into an onboarding checklist, measurably lifting week-1 connection rate +11%; stale indicator on the revenue widget only (finance users cared; others didn't). Storybook forcing states wired for all four widgets; QA added the blackhole-one-API pass. The quarterly audit six months later found one regression (a new widget shipped matrix-less, blank on empty) — one, versus the pre-discipline baseline of "all of them."

## Related skills
- `error-handling-strategy` — the code-side truth these states render.
- `form-design` — the form-specific error state discipline.
- `microcopy-writing` — the sentences inside every one of these states.
- `wireframe-to-spec` — where the state matrix becomes a deliverable.
