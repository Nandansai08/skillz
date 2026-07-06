# Install: OpenCode

OpenCode reads project instructions from `AGENTS.md` and supports referencing additional files from it.

```bash
git clone https://github.com/Nandansai08/skillz.git
mkdir -p .opencode/skills
cp skillz/skills/testing-qa/flaky-test-diagnosis/SKILL.md .opencode/skills/flaky-test-diagnosis.md
```

Reference from your project's `AGENTS.md`:

```markdown
## Skills

When a task matches, follow the workflow and Verification checklist in:
- .opencode/skills/flaky-test-diagnosis.md
```

Notes:

- Same context-budget advice as everywhere: import the router (`using-these-skills`) plus the handful of skills your project actually hits.
- OpenCode also supports custom commands; a skill's Workflow section converts naturally into one if you want slash-command invocation.
