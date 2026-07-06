# Install: Gemini CLI

Gemini CLI reads project context from `GEMINI.md` (and supports importing other markdown files).

```bash
git clone https://github.com/Nandansai08/skillz.git
mkdir -p .gemini/skills
cp skillz/skills/testing-qa/flaky-test-diagnosis/SKILL.md .gemini/skills/flaky-test-diagnosis.md
```

Then reference them from your `GEMINI.md`:

```markdown
# Project context

When a task matches one of these workflows, follow its steps and Verification checklist:

@./.gemini/skills/flaky-test-diagnosis.md
```

Notes:

- Import only the skills relevant to the project — everything imported rides in context.
- `skills/using-these-skills/SKILL.md` works well as the single always-imported router; import individual skills on demand.
