---
name: code-comments-that-last
description: >
  Use when writing or reviewing code comments and docstrings — deciding
  what deserves a comment, what will rot, and how to phrase constraints so
  they survive refactors. Triggers: "should I comment this", "add comments
  to", "review these docstrings", "this comment is outdated", "comment
  rot". NOT for user-facing API documentation (see api-documentation).
---

# Code Comments That Last

## Overview

A wrong comment is worse than none — readers trust comments over code exactly when they're debugging. Commenting only what code cannot express (whys, constraints, external anchors) keeps the comment layer small enough to stay true.

## When to Use

- Writing comments/docstrings for new code, or reviewing a PR heavy with them.
- Cleaning up a file where comments contradict the code.

**When NOT to use:**
- External-facing API documentation — `api-documentation`.

## Prerequisites

None.

## The Workflow

1. **Default to no comment; try to make the code say it first.** Rename the variable, extract a well-named function, add a type. A comment explaining *what* code does is a lint for unclear code.

2. **Comment the things code cannot express.** The durable categories:
   - **Why, not what:** `# Retry 3x: upstream returns spurious 503s during their daily failover (~02:00 UTC)`.
   - **Constraints and invariants:** `# Callers must hold self.lock — see flush()`.
   - **Non-obvious consequences:** `# Order matters: fee calc reads the discount set above`.
   - **External anchors:** spec section, RFC number, incident link, upstream bug URL.
   - **Deliberate deviations:** `# O(n²) is fine: n ≤ 16 by validation above` — this stops the "helpful" optimizer.

3. **Phrase comments to survive refactors.** Reference behavior and contracts, not line positions or names likely to change. "The function below" breaks when someone inserts a function; "callers of parse_row" doesn't.

4. **Delete rather than update-alongside.** When your change makes a comment wrong, the options are fix it or delete it — never leave it "roughly right." Comments are not API; wrong ones actively cause outages when a 3am debugger believes them.

5. **Docstrings: contract, not narration.** Document parameters only where the type doesn't say enough (units, valid ranges, ownership), return values with edge cases (`Returns None when the user has no active plan`), raised exceptions callers should catch, and side effects. Skip restating the signature.

6. **TODO hygiene.** Every TODO carries an owner or ticket: `# TODO(#4231): remove after v2 migration completes`. A bare `# TODO: fix this` is a wish, not a plan — in review, ask for the ticket or ask for the fix.

7. **Review pass:** for each comment in the diff ask (a) does the code already say this? → delete; (b) will this be true after the next plausible refactor? → rephrase per step 3; (c) is it explaining confusing code? → fix the code instead.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "More comments = better documented" | Comment volume correlates with rot surface, not clarity. Ten narration comments bury the one constraint comment that matters. |
| "I'll explain the change for the reviewer here" | `// changed to use map for performance` is PR-description content — it's noise the day after merge, and it rots the moment the next change lands. |
| "Leave the old code commented out, we might need it" | Git has it, forever, with context. The commented block just makes readers wonder if it's load-bearing. |
| "The docstring generator fills in all params — thorough!" | `user_id: the id of the user` is zero information with real maintenance cost: every signature change now has two places to edit and one of them silently lies. |
| "Someone might rely on that comment, better keep it" | Nothing may rely on a comment — that's what makes deleting wrong ones free and keeping them expensive. |
| "A section header comment organizes the file" | `# ---- helpers ----` is a file split refusing to happen. Structure belongs to modules, not banners. |

## Red Flags

- Comments narrating the diff ("added null check") — reviewer-directed noise.
- A comment contradicting the line below it — the rot already happened; nobody deleted.
- Docstrings restating every parameter name in sentence form.
- Bare TODOs with no ticket or owner, aging in place.
- Comments referencing "above"/"below"/"the next function" — position-coupled, one insert from wrong.
- A PR that changed behavior without touching the adjacent comment that describes the old behavior.

## Verification

- [ ] Every comment in the diff passes the three-question review pass (step 7) — reviewer confirms.
- [ ] No comment in the diff narrates what the adjacent code visibly does.
- [ ] Every TODO added carries a ticket/owner — grep the diff for `TODO` and check.
- [ ] Comments adjacent to changed behavior were updated or deleted — reviewer spot-checks the surrounding lines, not just the diff.
- [ ] Constraint comments phrase against behavior/contracts, not positions or transient names.

## Example

Before: `// increment i` / `// loop through users` / `// this is a hack, sorry`.
After review pass: the first two deleted (code says it); the third became
`// Hack: dedupe by email because upstream sends duplicate webhooks (their bug #881, promised fix Q3 2026). Remove when fixed — tracked in #2210.`
One comment survived, and it's the only one that will matter to the next reader.

## Related skills

- `api-documentation` — external-facing counterpart to docstrings.
- `code-review-checklist` — where the step-7 pass slots into a full review.
- `refactor-safely` — extracting functions as the alternative to explaining.
