# Roadmap

Direction, not dates (per this repo's own `roadmap-communication` skill: problems over promises).

## Now

- **Finish the anatomy retrofit** — 6 loop-engineering skills (`agent-loop-design`, `tool-use-loop-debugging`, `retry-and-backoff-strategy`, `feedback-loop-instrumentation`, `human-in-the-loop-checkpoints`, `convergence-criteria-design`) still carry the older section layout. Content is current; sections need the Rationalizations/Red Flags/Verification treatment.
- **Skill-anatomy linter** — a CI check validating new PRs against the CONTRIBUTING.md spec (frontmatter, section presence, name↔folder match, cross-link resolution), so format review stops being manual.

## Next

- **Marketplace manifest** — one-command install for Claude Code (`/plugin marketplace add` + `/plugin install`) instead of clone-and-copy.
- **More tool integrations** — deepen the per-tool docs beyond setup notes as each tool's rules/skills conventions stabilize; add tools with real demand.
- **Per-skill eval cases** — worked scenarios converted into testable eval inputs (the `eval-loop-for-agents` skill applied to this repo's own content).

## Later / open questions

- **Category expansion** — marketing was cut from v1 (weakest fit for agent workflows); add back if demand shows up in issues. Same bar for any new category: ≥6 genuinely process-shaped skills.
- **Should career-productivity / finance-ops stay?** They're practitioner workflows rather than code-agent skills. Community input welcome — see the `discussion` issue.
- **Translations** — only with maintainers who can hold quality per language.

Propose changes via issues; this file updates when direction actually changes.
