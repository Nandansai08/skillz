---
name: responsive-layout-strategy
description: >
  Use when planning how a UI adapts across screen sizes — breakpoints from
  content not devices, container queries, intrinsic layout with modern CSS,
  what changes at small sizes beyond shrinking. Triggers: "make this
  responsive", "breakpoint strategy", "mobile layout for this", "container
  queries", "the tablet view is broken", "responsive without media query soup".
---

# Responsive Layout Strategy

## When to use this skill
- Designing/building a UI that must work across viewport sizes, or refactoring a media-query jungle.
- Deciding component-level vs page-level adaptation.
- NOT for choosing native vs web or separate mobile products — this assumes one responsive surface; and not for pure visual design direction (`design-system-tokens` holds the vocabulary this layout consumes).

## Prerequisites
- Real content and real usage data (what devices actually visit — analytics beats device-market folklore), and the smallest/largest viewports you commit to supporting, written down.

## Workflow

1. **Let intrinsic layout do the work before ANY breakpoint exists:** modern CSS resolves most responsiveness with zero media queries —
   ```css
   .grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(280px, 1fr)); gap: var(--space-4); }
   .row  { display: flex; flex-wrap: wrap; }
   .measure { max-width: 65ch; }
   width: clamp(16rem, 50vw, 40rem);
   ```
   Auto-fit grids, wrapping flex, `minmax`, `clamp()`, `aspect-ratio` — components that size themselves to available space. Every breakpoint you don't write is one that can't be wrong at the in-between width nobody tested. Media queries become the exception layer for genuine *reorganization*, not the primary mechanism for sizing.

2. **Set breakpoints where the CONTENT breaks, not where devices live:** widen/narrow the real layout until it degrades (line lengths passing ~75ch, the sidebar starving the main column, cards stretching gaunt) — a breakpoint goes there, whatever pixel that is. Device-name breakpoints (`$tablet: 768px`) encode 2015's hardware into your CSS and guarantee the awkward in-betweens; content-driven sets are usually smaller (2–3 major reorganization points) and age better. Name them semantically (`--layout-narrow/wide`), not by device.

3. **Use container queries for components, media queries for page shells:** a Card in the main column vs the same Card in a sidebar needs different layouts at the same viewport — that's the container's width, not the viewport's:
   ```css
   .card-host { container-type: inline-size; }
   @container (min-width: 400px) { .card { grid-template-columns: 120px 1fr; } }
   ```
   Rule of thumb: page-level shell (nav placement, column count) responds to the viewport; everything inside responds to its container. This split is what makes components portable across layouts without per-context overrides — the design-system payoff.

4. **Treat small-screen as re-prioritization, not compression:** decide per screen what the phone-width user's top task is and lead with it; secondary content moves *down* (not into hamburger purgatory by default), data tables transform (priority columns + row-detail expansion, or cards — the 12-column table pinched to 320px serves nobody), navigation converts honestly (bottom bar for the 3–5 real destinations beats a drawer hiding everything equally). Mobile-first CSS ordering (base styles = narrow, enhance upward) keeps the cascade sane, but the design question is priority, not paint order.

5. **Handle the non-width dimensions that break real layouts:** touch targets and hover-dependence (`@media (hover: hover)` gating hover-revealed actions — the desktop-hover-only delete button is unreachable on touch; `accessibility-audit` step 5's 44px floor), viewport height (short landscape phones, the URL-bar dance — `dvh` units, and never pin critical actions under a `100vh` fold), zoom-as-resize (browser zoom to 200% IS a viewport change and must reflow, not clip — same audit, step 5), and `prefers-reduced-motion` on any layout animation.

6. **Test at the awkward widths, not the showcase ones:** the devtools drag from 320 to 1600 watching for the in-between uglies (the 700–900 band where desktop layouts half-fit is where unplanned layouts live), text zoomed 200%, a translation pass if localized (German labels are the layout's stress test — +35% string length), real device for touch/keyboard interplay. Storybook viewport addon or visual-regression shots at the *content* breakpoints ± 50px pins the behavior in CI.

7. **Encode the strategy in the system, not in per-page heroics:** the breakpoint set, container-query conventions, and table/nav transformation patterns documented in the design system (`design-system-tokens`' spacing scale feeding the gaps); new components required to demo at narrow/wide in review (the same forcing-function logic as `empty-loading-error-states` step 6). A responsive *strategy* is precisely the difference between 4 patterns reused and 40 pages each solving mobile alone.

## Common pitfalls
- Media-query soup: per-page pixel tweaks at 6 device breakpoints, unmaintainable and wrong between them — symptom of skipping intrinsic layout (step 1) and content-driven breaks (step 2).
- Desktop design "made responsive" by shrinking: 12 columns pinched, 11px text, hover-only affordances — mobile as an afterthought instead of a re-prioritization (step 4).
- Container-query-shaped problems solved with viewport queries: the sidebar Card broken because the *page* was wide even though the *slot* was narrow — then "fixed" with context-specific override classes, the portability killer (step 3).
- Testing only at 375 / 768 / 1440: the showcase widths pass while the 820px in-between — where a real chunk of traffic lives — renders the half-collapsed uglies (step 6).
- Hiding difficulty in the hamburger: "responsive" by relocating everything hard into a drawer, tanking discoverability of the product's second-most-important action.
- `100vh` heroes and pinned CTAs eaten by mobile browser chrome — the button that exists but can't be tapped (step 5's `dvh`).

## Example
Analytics dashboard "make it responsive" request; audit found 9 device-name breakpoints, per-page overrides, and a 14-column table pinched illegible at phone width (users emailing themselves CSVs to read on the train — the actual mobile use case surfaced by asking). Rework: layout shell to an auto-fit grid + one content breakpoint at 720px where the sidebar genuinely starved (found by dragging, not by device chart); widgets converted to container queries — the same RevenueCard now renders 2-col in main, stacked in sidebar, no overrides (deleted 340 lines of context CSS); the table transformed at narrow widths to priority-3-columns + expandable rows, chosen from the mobile task data ("check yesterday's number on the train" needs 3 columns, not 14). Hover-revealed row actions gated with `(hover: hover)` and given a touch affordance. Visual-regression shots at 670/720/770 pinned the breakpoint's neighborhood in CI. Result: mobile weekly-active on the dashboard 3.1× in a quarter — the train use case, finally served — and the next dashboard page shipped responsive in a day by consuming the patterns instead of inventing them.

## Related skills
- `design-system-tokens` — the spacing/type vocabulary the layouts consume.
- `accessibility-audit` — zoom, touch-target, and reflow requirements.
- `empty-loading-error-states` — the state matrix × the width matrix, same spec.
- `wireframe-to-spec` — capturing the reorganization decisions per breakpoint.
