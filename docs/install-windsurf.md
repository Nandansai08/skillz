# Install: Windsurf

Windsurf reads rules from `.windsurf/rules/` (per-project).

```bash
git clone https://github.com/Nandansai08/skillz.git
mkdir -p .windsurf/rules
cp skillz/skills/testing-qa/flaky-test-diagnosis/SKILL.md .windsurf/rules/flaky-test-diagnosis.md
```

Notes:

- Windsurf rules have activation modes (always-on / model-decision / glob). For skills, "model-decision" with the skill's `description` as the rule description mirrors this repo's intended routing.
- Windsurf enforces per-rule size limits — the skills here fit, but don't concatenate several into one rule file.
- Start with `using-these-skills` as an always-on rule; add task skills per project need.
