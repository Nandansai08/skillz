---
name: responsive-layout-strategy
description: >
  Use when planning how a UI adapts across screen sizes — breakpoints
  from content not devices, container queries, intrinsic layout with
  modern CSS, what changes at small sizes beyond shrinking. Triggers:
  "make this responsive", "breakpoint strategy", "mobile layout for
  this", "container queries", "the tablet view is broken", "responsive
  without media query soup". NOT for native-vs-web platform decisions or
  visual design direction.
---

# Responsive Layout Strategy

## Overview

Every breakpoint you don't write is one that can't be wrong at the in-between width nobody tested — intrinsic CSS resolves most responsiveness with zero media queries, content decides where the few real breakpoints go, and small-screen is a re-prioritization question, not a compression one.

## When to Use

- Designing/building a UI that must work across viewport sizes, or refactoring a media-query jungle.
- Deciding component-level vs page-level adaptation.

**When NOT to use:**
- Native-vs-web decisions; visual design direction (`design-system-tokens` holds the vocabulary).

## Prerequisites

- Real content and real usage data (what devices actually visit — analytics beats device folklore), and the smallest/largest supported viewports written down.

## The Workflow

1. **Let intrinsic layout do the work before ANY breakpoint exists:**
   ```css
   .grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(280px, 1fr)); gap: var(--space-4); }
   .row  { display: flex; flex-wrap: wrap; }
   .measure { max-width: 65ch; }
   width: clamp(16rem, 50vw, 40rem);
   ```
   Auto-fit grids, wrapping flex, `minmax`, `clamp()` — components that size themselves. Media queries become the exception layer for genuine *reorganization*, not the primary sizing mechanism.

2. **Set breakpoints where the CONTENT breaks, not where devices live:** widen/narrow the real layout until it degrades (line lengths past ~75ch, the sidebar starving the main column) — a breakpoint goes there, whatever pixel that is. Device-name breakpoints (`$tablet: 768px`) encode 2015's hardware and guarantee awkward in-betweens; content-driven sets are usually 2–3 reorganization points, named semantically.

3. **Use container queries for components, media queries for page shells:** a Card in the main column vs the same Card in a sidebar needs different layouts at the same viewport — that's the container's width:
   ```css
   .card-host { container-type: inline-size; }
   @container (min-width: 400px) { .card { grid-template-columns: 120px 1fr; } }
   ```
   Rule of thumb: the shell (nav placement, column count) responds to the viewport; everything inside responds to its container. This split is what makes components portable without per-context overrides — the design-system payoff.

4. **Treat small-screen as re-prioritization, not compression:** decide per screen what the phone-width user's top task is and lead with it; secondary content moves *down* (not into hamburger purgatory by default); data tables transform (priority columns + row expansion, or cards — the 12-column table pinched to 320px serves nobody); navigation converts honestly (bottom bar for the 3–5 real destinations beats a drawer hiding everything equally).

5. **Handle the non-width dimensions that break real layouts:** hover-dependence (`@media (hover: hover)` gating hover-revealed actions — the desktop-only delete button is unreachable on touch; plus `accessibility-audit`'s 44px targets), viewport height (`dvh` units; never pin critical actions under a `100vh` fold — mobile browser chrome eats it), zoom-as-resize (200% zoom must reflow, not clip), `prefers-reduced-motion` on layout animation.

6. **Test at the awkward widths, not the showcase ones:** drag 320→1600 watching for the in-between uglies (the 700–900 band where desktop layouts half-fit is where unplanned layouts live), text at 200%, a translation pass if localized (German labels are the stress test — +35% length). Visual-regression shots at the content breakpoints ±50px pin behavior in CI.

7. **Encode the strategy in the system, not per-page heroics:** the breakpoint set, container-query conventions, and table/nav transformation patterns documented in the design system; new components required to demo at narrow/wide in review (the same forcing-function logic as `empty-loading-error-states` step 6). The strategy is precisely the difference between 4 patterns reused and 40 pages each solving mobile alone.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "Start with the standard breakpoints: 768, 1024, 1440" | Device-named breakpoints encode dead hardware and guarantee the 820px uglies. Content decides where layout breaks; the standard set decides nothing. |
| "Shrink the desktop design proportionally — responsive done" | The pinched 12-column table and 11px text serve nobody. Small-screen is a priority question — what's the phone user's top task? — not a scale factor. |
| "Viewport queries work fine for the card component" | The sidebar Card breaks while the page is wide, then gets 'fixed' with context-override classes — the portability killer. Container width is the card's truth (step 3). |
| "Tested at mobile/tablet/desktop presets — covered" | The showcase widths pass while 820px — where real traffic lives — renders the half-collapsed layout nobody planned. The drag test finds what presets can't. |
| "Hamburger everything on mobile — standard pattern" | Relocating difficulty into a drawer tanks discoverability of the product's second action. Honest conversion (bottom bar for real destinations) beats uniform hiding. |
| "Hover-reveal keeps the UI clean" | And makes the delete button unreachable for every touch user. hover:hover gating plus a touch affordance — clean that excludes an input modality isn't clean. |

## Red Flags

- Per-page pixel tweaks at six device breakpoints.
- Components carrying context-specific override classes per placement.
- The 700–900px band visibly broken in a quick drag test.
- `100vh` heroes with pinned CTAs.
- Hover-only affordances with no touch path.
- German/longest-locale never rendered before ship.

## Verification

- [ ] Intrinsic-first: media-query count minimal and each one justified as reorganization — count noted.
- [ ] Breakpoints content-derived and semantically named — derivation shown (where the layout actually broke).
- [ ] Component adaptation via container queries; shell via viewport — spot-check the Card-in-sidebar case.
- [ ] Small-screen priority decisions documented per key screen; tables transformed, not pinched.
- [ ] Non-width checks pass: hover-gating, dvh, 200% zoom reflow, reduced-motion — walkthrough.
- [ ] Drag-test at awkward widths done; visual-regression shots at breakpoints ±50px in CI — links.

## Example

Analytics dashboard "make it responsive" request; audit found 9 device-name breakpoints, per-page overrides, and a 14-column table pinched illegible at phone width (users emailing themselves CSVs to read on the train — the actual mobile use case surfaced by asking). Rework: layout shell to an auto-fit grid + one content breakpoint at 720px where the sidebar genuinely starved (found by dragging, not by device chart); widgets converted to container queries — the same RevenueCard now renders 2-col in main, stacked in sidebar, no overrides (deleted 340 lines of context CSS); the table transformed at narrow widths to priority-3-columns + expandable rows, chosen from the mobile task data. Hover-revealed row actions gated with `(hover: hover)`. Visual-regression shots at 670/720/770 pinned the breakpoint's neighborhood in CI. Result: mobile weekly-active on the dashboard 3.1× in a quarter — the train use case finally served — and the next dashboard page shipped responsive in a day by consuming the patterns.

## Related skills

- `design-system-tokens` — the spacing/type vocabulary the layouts consume.
- `accessibility-audit` — zoom, touch-target, and reflow requirements.
- `empty-loading-error-states` — the state matrix × the width matrix, same spec.
- `wireframe-to-spec` — capturing the reorganization decisions per breakpoint.
