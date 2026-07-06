# How this repo compares to other skill libraries

Honest, non-marketing. If another library fits your need better, use it.

## Credit first

The skill anatomy this repo uses — most distinctively the **Common Rationalizations** table, **Red Flags**, and **evidence-based Verification** checklists — was adapted from [addyosmani/agent-skills](https://github.com/addyosmani/agent-skills). That repo originated the insight these sections encode: an agent's main failure mode isn't ignorance of the process, it's talking itself out of following the process, so a skill should preempt the excuses. We adopted the format because it's right, and we credit it because it isn't ours.

## vs addyosmani/agent-skills

| | agent-skills | this repo (skillz) |
|---|---|---|
| **Scope** | Software engineering craft, deep — TDD, debugging, refactoring, planning, phase-organized SDLC | 14 categories, wide — engineering plus data, security, SRE, product, design, research, finance, career, and agent-loop engineering |
| **Distinctive category** | The SDLC phase organization and the meta-skills around process discipline | `loop-engineering` — 10 skills on designing/debugging agentic loops (eval harnesses, context budgets, cost guardrails, HITL checkpoints), which theirs doesn't cover |
| **Depth style** | Fewer skills, each battle-tested around one engineering philosophy | 117 skills, each with a worked example; broader means some domains are one skill deep where a specialist library would have five |
| **Format** | The origin of the rationalizations/red-flags/verification anatomy | The same anatomy, adapted (we added per-skill worked Examples and NOT-for routing exclusions in descriptions) |

**Reach for theirs when:** your work is primarily software engineering process (TDD discipline, systematic debugging, planning) and you want the original, deepest treatment of it.

**Reach for this when:** your agent's tasks cross into data/PM/design/research/ops territory, or you're building agentic systems and want the loop-engineering category.

**Use both:** they compose — the formats are deliberately compatible, and there's no overlap penalty beyond context budget.

## vs anthropics/skills (official Anthropic skills)

Anthropic's official repo ships skills for document/artifact production (docx, pptx, xlsx, pdf) and platform-specific capabilities. Those are tool-shaped skills — they teach the agent to operate a capability. This repo's skills are process-shaped — they teach the agent how an expert practitioner works through a class of problem. Different layers; no competition. Use theirs for "make me a spreadsheet," this for "is this A/B test result trustworthy."

## vs prompt collections (awesome-prompts and kin)

Prompt collections are single-shot instructions optimized for one good output. Skills here are multi-step processes with exit criteria — the difference is process over prose, workflows over reference, steps with verification over essays without it. A prompt tells the model what to produce; a skill tells it how to work, including how to know it isn't done yet.

## What this repo doesn't do

- No tool-integration skills (no "how to use the GitHub MCP") — process only.
- No runnable scaffolding beyond a few templates — most skills are pure markdown workflow.
- Career/finance/PM categories are practitioner workflows, useful to agents assisting with that work — but they're not code-agent skills, and if that's all you need, a slimmer library serves better.
- Six of the loop-engineering skills still carry the older section anatomy (retrofit in progress — tracked in issues).
