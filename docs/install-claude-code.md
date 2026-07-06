# Install: Claude Code

## Manual (works today)

Clone and copy the skills you want:

```bash
git clone https://github.com/Nandansai08/skillz.git

# per-project
mkdir -p .claude/skills
cp -r skillz/skills/testing-qa/flaky-test-diagnosis .claude/skills/

# global (all projects)
cp -r skillz/skills/sre-incident-response/incident-triage ~/.claude/skills/
```

Copy a whole category by copying each skill folder inside it (see the gotcha below), or take everything:

```bash
# flatten all skills into ~/.claude/skills/
find skillz/skills -mindepth 2 -maxdepth 2 -type d -exec cp -r {} ~/.claude/skills/ \;
cp -r skillz/skills/using-these-skills ~/.claude/skills/
```

Start with `using-these-skills` — it's the router for everything else.

## ⚠️ The common snag: category nesting

Claude Code discovers skills at `.claude/skills/<skill-name>/SKILL.md`. This repo nests skills one level deeper (`skills/<category>/<skill-name>/`). If you copy a **category** folder:

```bash
cp -r skillz/skills/testing-qa .claude/skills/     # ✗ wrong
# yields .claude/skills/testing-qa/flaky-test-diagnosis/SKILL.md — one level too deep, not discovered
```

Copy the **skill** folders themselves (as in the examples above), or use the `find` one-liner to flatten.

## Plugin / marketplace

A marketplace manifest for one-command install (`/plugin install`) is on the roadmap — see the pinned roadmap issue. Until then, manual copy is the path.
