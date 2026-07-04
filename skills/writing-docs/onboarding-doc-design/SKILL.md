---
name: onboarding-doc-design
description: >
  Use when creating docs for new team members — day-1 environment setup,
  first-PR path, who-to-ask maps, and keeping onboarding docs from
  rotting. Triggers: "onboarding docs", "new hire setup guide", "document
  our dev setup", "first week guide", "new engineers keep getting stuck".
  NOT for the public project README (see readme-authoring) — onboarding
  docs assume org membership and cover access, people, and unwritten
  norms.
---

# Onboarding Doc Design

## Overview

A new person's only question is "what should I do *now*?" — so onboarding docs are a timeline, not a topic wiki. The flywheel is the design's engine: every new hire audits the docs by following them literally, and their stuck-points are the next hire's fixes.

## When to Use

- A new person is joining (the forcing function) or the last one's onboarding was archaeology.
- Converting tribal setup knowledge into something self-serve.

**When NOT to use:**
- Public project README — `readme-authoring`.

## Prerequisites

- A recent memory of what actually confused the last new person (or better: their notes — step 7's flywheel).

## The Workflow

1. **Structure by timeline, not by topic:** Day 1 (access + machine + hello-world), Week 1 (first PR shipped), Month 1 (context: architecture, on-call, norms). Topic-organized wikis make the new person assemble their own curriculum on their most overwhelmed day. Each phase ends with a concrete, verifiable milestone.

2. **Day-1 doc is a checklist with an unambiguous finish line:** accounts/access to request (with the *how* and the approver — "ask #it-help for GitHub org invite, ~2h turnaround"), machine setup as copy-paste commands (or better: one bootstrap script the doc merely runs — `environment-parity` step 5's containers cut this doc in half), and the finish line: "you're done when `make dev` serves localhost:3000 and the test suite passes." Include expected durations — knowing access requests take a day converts anxiety into scheduling.

3. **Week-1 centers on shipping a real-but-small first PR.** Maintain a `good-first-task` label with genuinely small, real items (a starter task that ships to production teaches the entire pipeline; no sandbox exercise does). Document the path explicitly: branching, review norms, who reviews, deploys, what to do when CI fails.

4. **Build the who-to-ask map — the highest-value, most-neglected page:** per system: owning team, the 1–2 humans with deep context, the right channel with its norms. Plus the social-permission line every new person needs in writing: "asking after 30 minutes of being stuck is the norm here, not a failure." CODEOWNERS + git-log frequency (`monorepo-navigation` step 3) generate the first draft — then correct by hand; the metadata rots.

5. **Month-1 layer is context, linked not duplicated:** architecture overview (one diagram + a request's life), the ADR log's greatest hits (`architecture-decision-record` — the five decisions everyone eventually asks "why?!" about), glossary of internal vocabulary, team rituals (standup, review SLAs, incident process).

6. **Make rot visible and cheap to fix:** every page carries an owner and last-verified date; setup docs re-verified on schedule OR replaced by the bootstrap script CI runs weekly (docs that are code can't silently rot). Dead pages deleted, not left as traps: a wrong onboarding doc costs more than a missing one, because the new person can't tell wrong from right yet.

7. **Run the flywheel: every new hire is the audit.** Their explicit day-1 assignment: "follow the docs literally; note every stuck-point and improvisation; your first-week PR can be the doc fixes." This is the fresh-clone test with a human — it finds the implicit steps veterans can't see, gives an immediate real contribution, and keeps the docs never more than one hire stale.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "We have a comprehensive wiki — onboarding is covered" | Forty unordered pages and the new person still DMs 'where do I start?' Coverage isn't the gap; sequence is. The timeline is the product. |
| "Setup is documented — I wrote it from my machine" | Your machine has the env var from 2023 and the implicit brew install. Only the follow-it-literally audit (step 7) sees what your configured environment hides. |
| "Their first task: add yourself to team.md — nice and easy" | Teaches nothing about the real pipeline. The genuinely-small REAL task ships through branch/review/CI/deploy — that's the onboarding exam the docs teach to. |
| "People will ask when they're stuck — we're friendly" | Without the written 30-minute norm, new people either bottleneck one buddy or sit stuck for three days silently. Social permission is a doc artifact, not a vibe. |
| "The doc describes our ideal process — aspirational is fine" | The new person follows the doc, collides with reality, and learns to distrust ALL docs at once. Document the actual process; fix the process separately. |
| "Keep the old setup page — some of it might still apply" | The new person can't tell which half is the wrong half. Wrong docs beat missing docs at generating confident failures; delete or fix, never leave the trap. |

## Red Flags

- Time-to-first-PR measured in weeks and nobody tracking it.
- The same setup question asked by three consecutive hires.
- Who-to-ask knowledge living in one buddy's DMs.
- Pages with no owner, no last-verified date.
- The onboarding docs' last commit predating the last architecture change.
- New-hire stuck-point notes requested by nobody.

## Verification

- [ ] Timeline structure with per-phase finish lines — doc linked.
- [ ] Day-1 checklist has approver + turnaround per access item; bootstrap script exists and CI-runs — links.
- [ ] `good-first-task` label populated with ≥3 real, small items — link.
- [ ] Who-to-ask map generated then hand-corrected — page linked; the 30-minute norm stated in writing.
- [ ] Owners + last-verified dates on every page — spot-check.
- [ ] Last hire's stuck-point audit completed and fixes merged — PR linked; time-to-first-PR recorded.

## Example

Team of 9, onboarding = "pair with whoever's free," last hire took 11 days to first PR. Build: Day-1 checklist with access table (owner + turnaround each) and a bootstrap script extracted from three people's contradictory setup notes — writing the script surfaced that two engineers were running different Postgres majors (`environment-parity` finding, fixed); finish line = green test suite. Week-1: `good-first-task` seeded with 6 curated items; review norms written by the actual most-frequent reviewer. Who-to-ask map generated from CODEOWNERS + git-log frequency, then corrected by hand (two listed owners had left — exactly why the map beats the metadata). Next hire: first PR on day 3 — and it was doc fixes from their stuck-points list (4 items, including one missing `xcode-select` step invisible to every Linux-using veteran). The hire after that: day 2, two stuck-points. The flywheel converging is the metric.

## Related skills

- `readme-authoring` — the public-facing cousin; fresh-clone test shared.
- `environment-parity` — the bootstrap script's foundation.
- `monorepo-navigation` — generating the ownership map's first draft.
- `docs-information-architecture` — organizing the month-1 layer.
