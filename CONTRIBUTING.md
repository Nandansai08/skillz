# Contributing

Thanks for contributing a skill. This document is the format spec. PRs that don't follow it will be asked to revise.

## What is a skill?

A skill is a folder containing a `SKILL.md` file that teaches Claude a repeatable, best-practice workflow for one specific task. Claude reads the `description` in the frontmatter to decide when to load the skill, then follows the body as expert guidance.

## Directory layout

```
skills/<category>/<skill-name>/
├── SKILL.md          # required
├── templates/        # optional: fill-in-the-blank boilerplate
├── scripts/          # optional: runnable helper scripts
└── reference/        # optional: lookup tables, cheat sheets
```

- Folder name is kebab-case and **must match** the `name` field in frontmatter.
- One skill = one job. If your SKILL.md needs an "or alternatively" fork into a different workflow, that's two skills.

## SKILL.md format

### Frontmatter (required)

```yaml
---
name: skill-name
description: >
  Use when [situation]. [What the skill does, one sentence.]
  Triggers: "phrase a user would say", "another phrase", relevant keywords.
---
```

The `description` is the **only thing Claude sees when deciding whether to load the skill**. Rules:

1. Lead with *when* to use it, not what it is. "Use when a test passes locally but fails in CI" beats "A skill about flaky tests."
2. Include literal trigger phrases a user would actually type.
3. State exclusions if the skill is easily confused with a sibling ("for unit tests; for end-to-end flakiness see `e2e-test-triage`").
4. Keep it under ~80 words. It's a routing signal, not a summary.

### Body sections (required, in this order)

```markdown
# Skill Title

## When to use this skill
Bullet list of concrete situations. Include a "NOT for..." bullet when
the skill has a common misuse.

## Prerequisites
What must exist or be known before starting (access, files, context).
Write "None" if truly none.

## Workflow
Numbered steps. Each step is an action with enough detail to execute —
commands, decision criteria, thresholds. This is the core of the skill.

## Common pitfalls
Real mistakes practitioners make, not generic warnings. "Forgetting to
X" is only a pitfall if you explain what breaks when you forget.

## Example
A concrete before/after, input/output, or worked scenario. Small but real.

## Related skills
Links to sibling skills with one clause on how they differ or connect.
Use the skill name in backticks: `other-skill-name`.
```

## Quality bar

- **Specific over general.** "Run `git bisect start; git bisect bad HEAD; git bisect good v1.2`" not "use bisection to narrow down the commit."
- **Opinionated.** A skill that lists five options without picking one isn't a skill, it's a survey. Recommend a default; note when to deviate.
- **Honest about limits.** Say what the workflow doesn't handle.
- **No filler.** Cut "it's important to note", "in today's world", restated headings. Every sentence should change what the reader does.

## Supporting files

- `templates/` — files meant to be copied and filled in (e.g. an ADR template). Reference them from the Workflow section by relative path.
- `scripts/` — small, dependency-free where possible, with a usage comment at the top.
- `reference/` — static lookup content (tables, cheat sheets) too long for the SKILL.md body.

Don't add supporting files a reader wouldn't actually copy or run.

## Submitting

1. Fork, create a branch: `skill/<skill-name>`.
2. Add your skill folder under the right category. New category? Open an issue first.
3. Check: frontmatter parses as YAML, folder name matches `name`, all six body sections present.
4. Open a PR using the template. One skill per PR preferred.

## Naming conventions

- kebab-case, 2–4 words, verb-or-noun phrase describing the job: `flaky-test-diagnosis`, `incident-triage`.
- No `claude-`, `ai-`, `how-to-` prefixes.
- No abbreviations unless universal (`api`, `sql`, `ci`).
