---
name: readme-authoring
description: >
  Use when writing or fixing a project README — answering what/why/
  quickstart in the first screen, ordering sections by reader need,
  keeping it honest as the project evolves. Triggers: "write a README",
  "improve this README", "document this project", "nobody can get started
  with our repo", "README review". NOT for full documentation sites (see
  docs-information-architecture) — the README is the front door, not the
  house.
---

# README Authoring

## Overview

A README's core claim is "this works, and you can have it running in five minutes" — every screen between the reader and that proof costs adoption. The fresh-clone test is the only honest verification, because authors' machines lie by omission.

## When to Use

- Creating a README for a new project or overhauling one that's grown stale/bloated.
- A steady stream of "how do I even run this?" questions signals the README is failing.

**When NOT to use:**
- Full documentation sites — `docs-information-architecture`.

## Prerequisites

- A working install/run path you have personally executed *from scratch* recently — you're about to sign it.

## The Workflow

1. **Nail the first screen: what, who, why-this-one.** Before any scrolling: one sentence saying what the project does in concrete terms ("CLI that syncs Postgres schemas between environments" — not "a modern, blazing-fast solution for schema workflows"), who it's for, and one honest sentence on when to use it vs the obvious alternative. Badges: keep the 2–3 that carry signal; delete the decoration row.

2. **Quickstart within the first scroll, runnable in under 5 minutes:** install command, minimal working example, expected output — shown, not described:
   ```
   $ pipx install schemasync
   $ schemasync diff --from staging --to prod
   ✓ 3 tables differ: users (+2 cols), orders (index), events (new)
   ```
   The expected-output block matters most: it's how readers verify success and discover what the tool actually produces. One golden path only — alternatives (Docker, from-source) go in a collapsed section or linked page.

3. **Order the remaining sections by reader frequency, not author pride:** usage examples for the 3–5 most common tasks → configuration (the 5 options people actually set, link the exhaustive reference) → troubleshooting the known gotchas → contributing/license links. The architecture essay and roadmap live in linked docs.

4. **Write examples against real use cases, tested.** Each example: the task in one line, the command/code, the output. Examples are the most-copied part of any README — a broken one is a support-ticket generator. Best practice: examples executed in CI (doctest-style, or a script running the fenced blocks) so they can't silently rot.

5. **State the honest boundaries:** requirements (runtime versions, OS support — tested ones, not hoped ones), project status (actively maintained / stable / experimental), and what it deliberately doesn't do. "Not supported: MySQL (see #42)" saves ten issues and builds more trust than silence.

6. **Audit pass before shipping — the fresh-clone test:** on a machine (or container) without your dev setup, follow the README literally, top to bottom. Every deviation you're forced to make — the implicit `npm install`, the env var you forgot existed, the system dependency — is a defect. This finds defects every single time it's run.

7. **Keep it maintained by keeping it small:** README changes ride in the same PR as the feature that invalidates them; a quarterly skim for rot (versions, screenshots, dead links — screenshots rot fastest, prefer text output blocks); and resist accretion — new first-screen content earns its place by evicting something.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "The setup is obvious to anyone who'd use this" | The fresh-clone test finds implicit steps every single time — the .npmrc copy, the system dep, the first-run migration. 'Obvious' is what your configured machine whispers to you. |
| "More install options = more welcoming" | Ten equal-billing install methods is choice paralysis in the doorway. One golden path converts; the alternatives serve the 5% from a collapsed section. |
| "Blazing-fast, batteries-included — sells the project" | Marketing adjectives spend the first screen saying nothing checkable, and technical readers discount everything after them. Concrete noun + verb; readers supply the adjectives. |
| "I'll show the command; the output is obvious" | Without the expected-output block, readers can't distinguish success from failure — and half the tool's value proposition (what it produces) stays invisible. |
| "Listing what we don't support looks weak" | 'Not supported: MySQL (#42)' prevents ten duplicate issues and reads as competence. Silence reads as either omission or ambush, both worse. |
| "We'll update the README in a follow-up PR" | The follow-up never lands, the v1 config example meets the v2 parser, and every new user's first experience is an error. Same-PR or it rots. |

## Red Flags

- Setup questions recurring in chat that the README claims to answer.
- Examples showing input but never output.
- A quickstart that has never been executed on a clean machine.
- Version requirements stated nowhere; the crash on old Node discovered by users.
- 2,000-line README with a hand-maintained TOC (a docs site refusing to be born).
- Screenshots of terminal output instead of text blocks.

## Verification

- [ ] First screen answers what/who/why-this-one in concrete terms — no marketing adjectives.
- [ ] Quickstart executed via fresh-clone/container test — deviations fixed; test date noted in the PR.
- [ ] Every example shows expected output; examples wired to CI where feasible — link.
- [ ] Requirements and not-supported list present and current.
- [ ] README diff included in the same PR as any invalidating feature change — spot-check recent history.

## Example

Internal tool, 40 stars, weekly "can't get it running" messages. Audit: first screen was a 12-badge row + feature list; install instructions assumed a private npm registry login nobody documented; the one example showed no output; Node version requirement (18+, it crashed on 16) unstated. Rewrite: one-sentence what/who, quickstart with the registry-auth step spelled out and expected output shown, requirements block (Node ≥18, tested on 18/20/22), "not supported: Windows without WSL (#88)" stated, architecture essay moved to `docs/`. Fresh-clone test in a container caught two more implicit steps (a `.npmrc` copy and a first-run migration). Result: setup questions dropped from ~4/week to ~1/month, and the next new hire onboarded without a single DM.

## Related skills

- `docs-information-architecture` — when the project outgrows the single file.
- `api-documentation` — the reference layer the README links to.
- `onboarding-doc-design` — the internal-team sibling of this skill.
- `changelog-writing` — the README's "what changed" companion.
