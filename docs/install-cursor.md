# Install: Cursor

Cursor reads project rules from `.cursor/rules/`. Skills are plain markdown, so they work as rules files:

```bash
git clone https://github.com/Nandansai08/skillz.git
mkdir -p .cursor/rules

# copy the SKILL.md files you want, renamed per skill
cp skillz/skills/testing-qa/flaky-test-diagnosis/SKILL.md .cursor/rules/flaky-test-diagnosis.md
cp skillz/skills/using-these-skills/SKILL.md .cursor/rules/using-these-skills.md
```

Notes:

- Cursor's rules don't use the YAML `description` for auto-routing the way Claude Code does — for always-relevant guidance (like `using-these-skills`), consider Cursor's "Always" rule type; for task-specific skills, "Agent Requested" with the description pasted into the rule metadata works well.
- Copy a handful of skills relevant to your project rather than all 117 — rules share the context budget.
- Skills referencing `templates/` files: copy those alongside or inline the template.
