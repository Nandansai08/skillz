---
name: design-system-tokens
description: >
  Use when creating or restructuring design tokens — color/type/spacing
  scales, semantic naming layers, theming (dark mode), and keeping tokens
  synchronized between design tools and code. Triggers: "set up design
  tokens", "token naming", "dark mode theming", "our colors are a mess",
  "spacing scale", "design system variables".
---

# Design System Tokens

## When to use this skill
- Establishing tokens for a design system, or rescuing a codebase with 45 grays.
- Adding theming (dark mode, brands) to an existing UI.
- NOT for component API design or which components to build — tokens are the vocabulary beneath components; this skill stops at the token layer.

## Prerequisites
- An inventory of current values in the wild: `rg "#[0-9a-fA-F]{3,8}" --type css -o | sort | uniq -c | sort -rn` (and the equivalent for spacing/font sizes) — the mess quantified is the mandate and the migration map.

## Workflow

1. **Build the two-layer architecture — primitive and semantic — and enforce the direction:**
   - **Primitives:** the raw palette, named by what they ARE: `blue-500`, `gray-100`, `space-4`, `text-lg`. Exhaustive, boring, never referenced directly by product code.
   - **Semantic tokens:** named by ROLE, pointing at primitives: `color-text-primary → gray-900`, `color-bg-danger → red-600`, `space-inset-card → space-4`.
   Product code consumes semantic only. This single rule is what makes theming possible (dark mode remaps semantic→primitive bindings, zero component changes) and what makes "change our danger red" a one-line edit instead of a 200-file grep.

2. **Name semantics by a consistent grammar, decided once:** a fixed order like `[category]-[role]-[modifier?]-[state?]` — `color-text-secondary`, `color-bg-interactive-hover`, `border-danger`. The grammar test: can someone who's never seen the token list *predict* the name they need? Names describing appearance (`color-light-blue-bg`) are primitives leaking upward — they break the moment the theme changes the value (the `light-blue` token that's dark navy in dark mode).

3. **Constrain the scales deliberately — tokens are a budget, not a catalog:**
   - **Spacing:** one geometric-ish scale (4/8-based: 4, 8, 12, 16, 24, 32, 48, 64) — enough steps for rhythm, few enough that "which one?" has an answer.
   - **Type:** a modular scale of 6–9 sizes with paired line-heights (composite tokens: `text-body` = size+leading+weight together — separately-mixable font attributes recreate the chaos).
   - **Color primitives:** per hue, a lightness ramp (50–900) generated with perceptual uniformity (OKLCH-based tooling) so `-500` means the same visual weight across hues, and contrast pairs can be *designed into the ramp* (every `-600+` passes AA on white — `accessibility-audit`'s contrast findings, prevented at the source).
   The inventory from prerequisites maps onto these scales; the 45 grays usually collapse to 8.

4. **Implement as CSS custom properties (or platform equivalent), themed at the binding layer:**
   ```css
   :root { --color-text-primary: var(--gray-900); }
   [data-theme="dark"] { --color-text-primary: var(--gray-50); }
   ```
   Dark mode = a re-binding of semantic tokens, NOT `filter: invert`, NOT per-component overrides, and NOT simply flipping the ramp (dark themes need desaturated accents and *reduced* contrast ratios in places — elevated surfaces get lighter, shadows become glows; budget design time for the dark bindings as their own pass).

5. **Single source of truth, transformed outward:** tokens defined once (W3C design-tokens JSON / Style Dictionary / Tokens Studio) and generated into every target — CSS vars, Tailwind config, iOS/Android, Figma variables. The failure mode being prevented: Figma says `gray-800`, code says `#2d2f33`, and "is this the right gray?" becomes a Slack thread per PR. Sync direction and cadence explicit (design-tool → repo via PR, reviewed like code).

6. **Migrate by strangling, not big-bang:** semantic layer shipped first; a lint rule (stylelint declaration-strict-value or equivalent) rejecting NEW raw hex/px in components ("no new debt"); existing hardcodes converted opportunistically per-file-touched, tracked with the prerequisite grep count as the burndown metric. The big-bang restyle branch dies in review conflicts; the ratchet ships.

7. **Govern additions with a threshold, prune on a cadence:** new-token requests need a role that ≥2 use sites share ("this exact orange for this one banner" is a component style, not a token); quarterly, count usages per token (static analysis) and merge/deprecate the near-orphans — token sets grow toward entropy exactly like the hexes they replaced, and 300 semantics with 40 used is the old problem wearing better names.

## Common pitfalls
- One-layer systems: product code referencing `blue-500` directly everywhere — theming now means touching every file, and the primitive rename is a breaking change. The two layers ARE the system.
- Appearance-named "semantics" (`--light-gray-border`) — dark mode arrives and the name lies forever after (step 2's grammar test catches these at review).
- Scale sprawl reintroduced through the side door: `space-4-and-a-half`, `text-15px`, "just this once" — the lint ratchet (step 6) is the only "no" that holds under deadline.
- Dark mode as arithmetic (auto-inverted ramps, unchanged accent saturation) — technically themed, visually broken; the dark binding pass is design work (step 4).
- Figma and code drifting because sync is manual and someone's on vacation — if the transform isn't automated, the source of truth is whichever was updated last (step 5).
- Governance-free growth: every feature adds four tokens, none retire, and in 18 months the token picker is harder to search than the hex list was (step 7's threshold + pruning).

## Example
B2B app, redesign blocked on "add dark mode": inventory grep found 47 grays, 23 blues, 31 distinct spacing values across 400 components. Ramp design: grays collapsed to an 8-step OKLCH ramp with AA-on-white guaranteed from 600 up; semantic layer of 32 color roles + 8 spacing + 7 composite type tokens, grammar `category-role-state`. Two sev findings from the old world fixed structurally: disabled-text contrast (now a designed pair) and the four slightly-different card paddings (now one `space-inset-card`). Dark bindings done as a design pass (elevated surfaces lightened, accent desaturated 15% — the auto-inverted prototype had been rejected on sight, correctly). Style Dictionary pipeline → CSS vars + Tailwind + Figma variables; stylelint ratchet on raw values. Migration: burndown from 4,100 raw values to 240 over two quarters, opportunistic-per-PR. Dark mode shipped as a 190-line binding file, zero component edits — the sentence that justified the whole architecture to the skeptics.

## Related skills
- `accessibility-audit` — contrast designed into ramps beats contrast audited after.
- `responsive-layout-strategy` — the layout sibling of this vocabulary.
- `wireframe-to-spec` — specs that reference tokens instead of hexes.
- `release-versioning` — versioning the token package like the API it is.
