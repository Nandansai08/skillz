---
name: readme-authoring
description: >
  Use when writing or fixing a project README — answering what/why/quickstart
  in the first screen, ordering sections by reader need, keeping it honest
  as the project evolves. Triggers: "write a README", "improve this README",
  "document this project", "nobody can get started with our repo",
  "README review".
---

# README Authoring

## When to use this skill
- Creating a README for a new project or overhauling one that's grown stale/bloated.
- A steady stream of "how do I even run this?" questions signals the README is failing.
- NOT for full documentation sites — see `docs-information-architecture`; the README is the front door, not the house.

## Prerequisites
- A working install/run path you have personally executed *from scratch* recently — the README's core claim is "this works," and you're about to sign it.

## Workflow

1. **Nail the first screen: what, who, why-this-one.** Before any scrolling: one sentence saying what the project does in concrete terms ("CLI that syncs Postgres schemas between environments" — not "a modern, blazing-fast solution for schema workflows"), who it's for, and one honest sentence on when to use it vs the obvious alternative. Badges: keep the 2–3 that carry signal (CI, version); delete the decoration row.

2. **Quickstart within the first scroll, runnable in under 5 minutes:** install command, minimal working example, expected output — shown, not described:
   ```
   $ pipx install schemasync
   $ schemasync diff --from staging --to prod
   ✓ 3 tables differ: users (+2 cols), orders (index), events (new)
   ```
   The expected-output block matters most: it's how readers verify success and how they discover what the tool actually produces. One golden path only — alternatives (Docker, from-source, other OSes) go in a collapsed section or linked page.

3. **Order the remaining sections by reader frequency, not author pride:** usage examples for the 3–5 most common tasks → configuration (the 5 options people actually set, link the exhaustive reference) → troubleshooting the known 3 gotchas → contributing/license links. The architecture essay, benchmark methodology, and roadmap live in linked docs — every screen of README between a reader and their answer costs adoption.

4. **Write examples against real use cases, tested.** Each example: the task in one line, the command/code, the output. Examples are the most-copied part of any README — a broken one is a support ticket generator. Best practice: examples executed in CI (doctest-style, or a script that runs the README's fenced blocks) so they can't silently rot (`code-comments-that-last` logic at repo scale).

5. **State the honest boundaries:** requirements (runtime versions, OS support — tested ones, not hoped ones), project status (actively maintained / stable / experimental), and what it deliberately doesn't do. "Not supported: MySQL (see #42)" saves ten issues and builds more trust than silence.

6. **Audit pass before shipping — the fresh-clone test:** on a machine (or container) without your dev setup, follow the README literally, top to bottom. Every deviation you're forced to make — the implicit `npm install` step, the env var you forgot existed, the system dependency — is a defect. This is `runbook-authoring` step 6's cold-test, and it finds defects every single time.

7. **Keep it maintained by keeping it small:** README changes ride in the same PR as the feature that invalidates them (reviewers should ask "does the README still hold?"); a quarterly skim for rot (versions, screenshots, dead links — screenshots rot fastest, prefer text output blocks); and resist accretion — new content earns its place on the first screen by evicting something.

## Common pitfalls
- Marketing-voice openers ("blazingly fast, batteries-included, next-generation") that spend the first screen saying nothing checkable. Concrete noun + verb; the reader decides the adjectives.
- Quickstart that assumes the author's machine: undeclared system deps, a config file that "everyone has," Postgres already running. The fresh-clone test (step 6) exists for this.
- Ten install methods given equal billing — choice paralysis in the front door. One golden path, alternatives demoted.
- Examples showing input but never output — the reader can't tell success from failure, and half the tool's value proposition (its output) stays invisible.
- README as the only docs, swelling to 2,000 lines with a hand-maintained TOC — that's a docs site refusing to be born (`docs-information-architecture`).
- Silent rot: the v2 config format shipped, the README still shows v1, and every new user's first experience is a config error. The same-PR rule (step 7) is the antibody.

## Example
Internal tool, 40 stars, weekly "can't get it running" messages. Audit: first screen was a 12-badge row + feature list; install instructions assumed a private npm registry login nobody documented; the one example showed no output; Node version requirement (18+, it crashed on 16) unstated. Rewrite: one-sentence what/who, quickstart with the registry-auth step spelled out and expected output shown, requirements block (Node ≥18, tested on 18/20/22), "not supported: Windows without WSL (#88)" stated, architecture essay moved to `docs/`. Fresh-clone test in a container caught two more implicit steps (a `.npmrc` copy and a first-run migration). Result measured crudely but honestly: setup questions in the team channel dropped from ~4/week to ~1/month, and the next new hire onboarded without a single DM.

## Related skills
- `docs-information-architecture` — when the project outgrows the single file.
- `api-documentation` — the reference layer the README links to.
- `onboarding-doc-design` — the internal-team sibling of this skill.
- `changelog-writing` — the README's "what changed" companion.
