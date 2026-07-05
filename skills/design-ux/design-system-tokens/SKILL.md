---
name: design-system-tokens
description: >
  Use when creating or restructuring design tokens — color/type/spacing
  scales, semantic naming layers, theming (dark mode), keeping tokens
  synchronized between design tools and code. Triggers: "set up design
  tokens", "token naming", "dark mode theming", "our colors are a mess",
  "spacing scale", "design system variables". NOT for component API
  design or which components to build — tokens are the vocabulary beneath
  components.
---

# Design System Tokens

## Overview

Two layers make the whole system work: primitives named by what they ARE, semantics named by ROLE, and product code touching semantics only — which is what makes dark mode a re-binding file instead of a component rewrite. Tokens are a budget, not a catalog; the lint ratchet is the only "no" that holds under deadline.

## When to Use

- Establishing tokens for a design system, or rescuing a codebase with 45 grays.
- Adding theming (dark mode, brands) to an existing UI.

**When NOT to use:**
- Component APIs and inventory — a different layer, above this vocabulary.

## Prerequisites

- The current mess quantified: `rg "#[0-9a-fA-F]{3,8}" --type css -o | sort | uniq -c | sort -rn` (and equivalents for spacing/sizes) — the inventory is the mandate and the migration map.

## The Workflow

1. **Build the two-layer architecture — primitive and semantic — and enforce the direction:**
   - **Primitives:** raw palette, named by what they ARE: `blue-500`, `gray-100`, `space-4`. Exhaustive, boring, never referenced by product code.
   - **Semantic tokens:** named by ROLE, pointing at primitives: `color-text-primary → gray-900`, `color-bg-danger → red-600`.
   Product code consumes semantic only. This single rule is what makes theming possible (dark mode remaps bindings, zero component changes) and "change our danger red" a one-line edit.

2. **Name semantics by a consistent grammar, decided once:** `[category]-[role]-[modifier?]-[state?]` — `color-text-secondary`, `color-bg-interactive-hover`. The grammar test: can someone who's never seen the list *predict* the name they need? Appearance-named "semantics" (`color-light-blue-bg`) are primitives leaking upward — they lie the moment the theme changes the value.

3. **Constrain the scales deliberately — tokens are a budget, not a catalog:**
   - **Spacing:** one 4/8-based scale (4, 8, 12, 16, 24, 32, 48, 64).
   - **Type:** 6–9 sizes with paired line-heights as composite tokens (`text-body` = size+leading+weight together — separately-mixable attributes recreate the chaos).
   - **Color primitives:** per-hue lightness ramps (50–900) generated perceptually (OKLCH tooling) so `-500` means the same weight across hues, with contrast pairs *designed into the ramp* (every `-600+` passes AA on white — `accessibility-audit` findings prevented at the source).
   The 45 grays usually collapse to 8.

4. **Implement as CSS custom properties, themed at the binding layer:**
   ```css
   :root { --color-text-primary: var(--gray-900); }
   [data-theme="dark"] { --color-text-primary: var(--gray-50); }
   ```
   Dark mode = a re-binding of semantics — NOT `filter: invert`, and not a mechanical ramp-flip: dark themes need desaturated accents, lightened elevated surfaces, and adjusted contrast. Budget the dark bindings as a design pass of their own.

5. **Single source of truth, transformed outward:** tokens defined once (W3C design-tokens JSON / Style Dictionary / Tokens Studio), generated into every target — CSS vars, Tailwind config, iOS/Android, Figma variables. The failure being prevented: Figma says `gray-800`, code says `#2d2f33`, and "is this the right gray?" becomes a Slack thread per PR. Sync automated, or the source of truth is whichever was updated last.

6. **Migrate by strangling, not big-bang:** semantic layer shipped first; a lint rule rejecting NEW raw hex/px in components ("no new debt"); existing hardcodes converted per-file-touched, tracked with the inventory grep as the burndown metric. The big-bang restyle branch dies in review conflicts; the ratchet ships.

7. **Govern additions with a threshold, prune on a cadence:** new tokens need a role ≥2 use sites share ("this exact orange for this one banner" is a component style, not a token); quarterly usage counts merge/deprecate near-orphans — token sets grow toward entropy exactly like the hexes they replaced.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "One layer is simpler — just use blue-500 directly" | Direct-primitive consumption makes theming touch every file and every rename a breaking change. The two layers ARE the system; one layer is a nicely-named hex list. |
| "color-light-blue-bg describes exactly what it is" | Until dark mode makes it navy, and the name lies forever after. Role names survive theme changes; appearance names are future bugs with good intentions. |
| "Just this once — space-4-and-a-half for this component" | The side-door is how 31 spacing values happened the first time. The lint ratchet exists because 'just once' under deadline is the entropy mechanism itself. |
| "Dark mode = invert the ramp programmatically" | Auto-inverted dark themes ship with screaming saturated accents and backwards elevation. The bindings are a design pass; the arithmetic version gets rejected on sight (as the example's was). |
| "We'll sync Figma manually when things settle" | Things never settle, and the source of truth becomes whichever tool was touched last. Unautomated sync is drift with a schedule. |
| "More tokens = more flexibility for teams" | 300 semantics with 40 used is the old problem wearing better names. The ≥2-use-sites threshold and quarterly pruning keep the vocabulary a vocabulary. |

## Red Flags

- Product components referencing primitives (or raw hex) directly.
- Token names containing color words in the semantic layer.
- The token count growing monotonically, no pruning ever.
- Dark mode PRs touching component files en masse.
- Figma values and code values drifting (spot-check disagreements).
- New raw values merging with no lint complaint.

## Verification

- [ ] Two-layer architecture live: grep shows zero primitive/hex references in product components (or a burndown trending there) — counts attached.
- [ ] Grammar documented; new names predictable — spot-test with someone unfamiliar.
- [ ] Scales constrained with counts (spacing steps, type sizes, ramp stops) — listed.
- [ ] Contrast designed into ramps — AA pairs verified programmatically.
- [ ] Single-source pipeline generating all targets — config linked; Figma/code spot-check matches.
- [ ] Lint ratchet live in CI — a test PR with a raw hex shown failing.
- [ ] Theme = binding file only — the dark-mode diff shown touching zero components.

## Example

B2B app, redesign blocked on "add dark mode": inventory grep found 47 grays, 23 blues, 31 spacing values across 400 components. Ramp design: grays collapsed to an 8-step OKLCH ramp with AA-on-white guaranteed from 600 up; semantic layer of 32 color roles + 8 spacing + 7 composite type tokens. Two old-world findings fixed structurally: disabled-text contrast (now a designed pair) and four slightly-different card paddings (now one `space-inset-card`). Dark bindings done as a design pass (elevated surfaces lightened, accent desaturated 15% — the auto-inverted prototype had been rejected on sight, correctly). Style Dictionary pipeline → CSS vars + Tailwind + Figma variables; stylelint ratchet on raw values. Migration: burndown from 4,100 raw values to 240 over two quarters. Dark mode shipped as a 190-line binding file, zero component edits — the sentence that justified the whole architecture to the skeptics.

## Related skills

- `accessibility-audit` — contrast designed into ramps beats contrast audited after.
- `responsive-layout-strategy` — the layout sibling of this vocabulary.
- `wireframe-to-spec` — specs that reference tokens instead of hexes.
- `release-versioning` — versioning the token package like the API it is.
