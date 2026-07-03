---
name: onboarding-doc-design
description: >
  Use when creating docs for new team members — day-1 environment setup,
  first-PR path, who-to-ask maps, and keeping onboarding docs from rotting.
  Triggers: "onboarding docs", "new hire setup guide", "document our dev
  setup", "first week guide", "new engineers keep getting stuck".
---

# Onboarding Doc Design

## When to use this skill
- A new person is joining (the forcing function) or the last one's onboarding was archaeology.
- Converting tribal setup knowledge into something self-serve.
- NOT for the public project README (`readme-authoring`) — onboarding docs assume org membership and cover the parts a README never can: access, people, unwritten norms.

## Prerequisites
- A recent memory of what actually confused the last new person (or better: their notes — see step 7's flywheel).

## Workflow

1. **Structure by timeline, not by topic:** Day 1 (access + machine + hello-world), Week 1 (first PR shipped), Month 1 (context: architecture, on-call, norms). A new person's only question is "what should I do *now*?" — topic-organized wikis make them assemble their own curriculum on their most overwhelmed day. Each phase ends with a concrete, verifiable milestone.

2. **Day-1 doc is a checklist with an unambiguous finish line:** accounts/access to request (with the *how* and the approver — "ask #it-help for GitHub org invite, ~2h turnaround"), machine setup as copy-paste commands (or better: one bootstrap script the doc merely runs — `environment-parity` step 5's containers/devcontainers cut this doc in half), and the finish line: "you're done when `make dev` serves localhost:3000 and the test suite passes." Include expected durations — knowing access requests take a day converts anxiety into scheduling.

3. **Week-1 centers on shipping a real-but-small first PR.** Maintain a `good-first-task` label with genuinely small, real items (a starter task that ships to production teaches the entire pipeline: branch conventions, review norms, CI, deploy — no sandbox exercise does). Document the path explicitly: how we branch, what reviews look like here, who reviews, how deploys happen, what to do when CI fails. The first PR is the onboarding exam the docs are teaching to.

4. **Build the who-to-ask map — the highest-value, most-neglected page:** per system/domain: the owning team, the 1–2 humans who hold the deep context, and the right channel (with norms: "post in #platform-help, don't DM"). Plus the social-permission line every new person needs in writing: "asking after 30 minutes of being stuck is the norm here, not a failure." CODEOWNERS + `git log` frequency (`monorepo-navigation` step 3) generate the first draft of this map.

5. **Month-1 layer is context, linked not duplicated:** architecture overview (one diagram + prose walkthrough of a request's life), the ADR log's greatest hits (`architecture-decision-record` — the five decisions everyone eventually asks "why?!" about), glossary of internal vocabulary (codenames, acronyms, the misleadingly-named service everyone warns about verbally), team rituals (standup, review SLAs, incident process → `incident-triage`, `on-call-handoff` when they join the rotation).

6. **Make rot visible and cheap to fix:** every page carries an owner and last-verified date; setup docs get re-verified on a schedule OR — better — replaced by the bootstrap script that CI runs weekly (docs that are code can't silently rot; `runbook-authoring` step 7's discipline applies). Dead pages deleted, not left as traps: a wrong onboarding doc costs more than a missing one, because the new person can't tell wrong from right yet.

7. **Run the flywheel: every new hire is the audit.** Their explicit day-1 assignment: "follow the docs literally; every place you get stuck or improvise, note it; your first-week PR can be the doc fixes." This is the fresh-clone test (`readme-authoring` step 6) with a human — it finds the implicit steps veterans can't see, it gives the new person an immediate real contribution, and it means the docs are never more than one hire stale.

## Common pitfalls
- The 40-page wiki nobody assigned an order to — comprehensive, and the new person still DMs "where do I actually start?" Timeline structure is the fix, not more content.
- Setup docs written by someone with a working environment, from memory — the implicit `brew install` steps, the env var that's been in their shell since 2023. Only the step-7 audit catches these.
- First task that's a toy ("add your name to team.md") — teaches nothing about the real pipeline — or a landmine ("small" ticket touching the legacy auth module). Curate the label for real-and-genuinely-small.
- No who-to-ask map, so the new person either asks their one onboarding buddy everything (bottleneck) or asks nobody (three days stuck, silently).
- Docs describing the aspirational process, not the actual one ("we do trunk-based development" while every PR waits two days for review) — new people follow the doc, collide with reality, and learn to distrust all docs at once.
- Treating onboarding docs as HR's — the *team's* engineering docs are where the setup, systems, and norms live; HR covers benefits, not `make dev`.

## Example
Team of 9, onboarding = "pair with whoever's free," last hire took 11 days to first PR. Build: Day-1 checklist with access table (owner + turnaround each) and a bootstrap script extracted from three people's contradictory setup notes — writing the script surfaced that two engineers were running different Postgres majors (`environment-parity` finding, fixed); finish line = green test suite. Week-1: `good-first-task` label seeded with 6 curated items; the review-norms section written by the actual most-frequent reviewer. Who-to-ask map generated from CODEOWNERS + git-log frequency, then corrected by hand (two listed owners had left; the real experts were elsewhere — exactly why the map beats the metadata). Next hire: first PR on day 3 — and it was doc fixes from their stuck-points list (4 items, including one missing `xcode-select` step invisible to every Linux-using veteran). The hire after that: day 2, two stuck-points. The flywheel converging is the metric.

## Related skills
- `readme-authoring` — the public-facing cousin; fresh-clone test shared.
- `environment-parity` — the bootstrap script's foundation.
- `monorepo-navigation` — generating the ownership map's first draft.
- `docs-information-architecture` — organizing the month-1 layer.
