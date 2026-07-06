# Contributing

Thanks for contributing a skill. This document is the format spec and the submission process. PRs that don't follow it will be asked to revise.

## What is a skill?

A skill is a folder containing a `SKILL.md` that teaches an AI agent a repeatable, best-practice workflow for one specific task. The agent reads the frontmatter `description` to decide when to load the skill, then follows the body as a process — not as reference material to summarize.

## Pre-flight checklist (before opening a PR)

1. **Search the existing catalog for overlap.** Check the [README index](README.md) and grep `skills/` for your topic. Similar skill exists → extend it instead. **We prefer extending an existing skill over adding a near-duplicate.**
2. **Check open PRs** for the same or overlapping proposals.
3. **Confirm your draft fits the anatomy** below — all sections, in order.
4. **Justify in the PR description** why this isn't better served by extending an existing skill.

## Directory layout

```
skills/<category>/<skill-name>/
├── SKILL.md          # required
├── templates/        # optional: fill-in-the-blank boilerplate
├── scripts/          # optional: runnable helpers
└── reference/        # optional: lookup tables
```

- Folder name is kebab-case and **must match** the frontmatter `name`.
- One skill = one job. If your SKILL.md forks into "or alternatively" workflows, that's two skills.
- Don't create empty `scripts/`/`templates/`/`reference/` folders to mirror other skills — add them only where a skill genuinely needs a runnable helper or lookup table.

## SKILL.md anatomy

Every skill has these sections, in this order. Equivalent headings are fine if they serve the same purpose.

### Frontmatter

```yaml
---
name: skill-name
description: >
  Use when [situation]. Triggers: "phrase a user would type", keywords.
  NOT for [adjacent situation] (see sibling-skill).
---
```

The description is what an agent reads to decide whether to load the skill. Rules:

1. **Positive triggers AND explicit exclusions.** "Use when X" plus "NOT for Y" — the exclusions prevent wrong-skill loads as effectively as triggers cause right ones.
2. **Don't summarize the workflow in the description.** A summary invites following the summary instead of the real steps.
3. Include literal phrases a user would type.
4. Under ~90 words. It's a routing signal.

### Body sections

| Section | Content |
|---|---|
| `## Overview` | 1–2 sentences: what this does and why it matters. |
| `## When to Use` | Trigger bullets + a **When NOT to use** block with pointers to the right sibling. |
| `## Prerequisites` | What must exist/be known before starting. "None" if none. |
| `## The Workflow` | Numbered, executable steps — commands, thresholds, decision criteria. Code examples where useful; ASCII flowcharts at real decision points. Technique-warnings inline at the step where the mistake happens. |
| `## Common Rationalizations` | Two-column table: excuse an agent might use to skip a step ↔ why it doesn't hold. The most distinctive section — it preempts the agent talking itself out of the process. |
| `## Red Flags` | Observable signs the process is being skipped or violated — for self-monitoring and for humans reviewing agent work. |
| `## Verification` | Checklist of exit criteria, each evidence-based (passing output, build result, a concrete artifact) — never "looks right." The task isn't done until this passes. |
| `## Example` | A concrete worked scenario with real numbers. |
| `## Related skills` | Sibling links with one clause on how each differs or connects. |

## Quality bar

- **Specific over general.** "Run `git bisect run ./check.sh`" not "use bisection."
- **Opinionated.** Recommend a default; note when to deviate. Five options without a pick is a survey.
- **Honest rationalizations.** The table's excuses should be ones a capable agent would actually generate, paired with reasons that survive scrutiny — not strawmen.
- **Evidence-based verification.** Every checklist item names its artifact. "Tests pass" → "failing-then-passing output linked."
- **No filler.** Every sentence should change what the reader does.

## Format credit

This anatomy — particularly Common Rationalizations, Red Flags, and evidence-based Verification — is adapted from [addyosmani/agent-skills](https://github.com/addyosmani/agent-skills). See [docs/comparison.md](docs/comparison.md) for how the two repos differ.

## Submitting

1. Fork; branch `skill/<skill-name>`.
2. Run the pre-flight checklist above.
3. Validate: frontmatter parses as YAML, folder matches `name`, all sections present.
4. Open a PR using the template. One skill per PR.

## Naming

kebab-case, 2–4 words, the job's name: `flaky-test-diagnosis`, `incident-triage`. No `ai-`/`how-to-` prefixes.
