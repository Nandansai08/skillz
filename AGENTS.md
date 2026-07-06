# AGENTS.md — instructions for AI agents working on this repository

This repo is a library of agent skills. If you're an agent making changes here, this file is your contract.

## What this repo is

117+ skills under `skills/<category>/<skill-name>/SKILL.md`, each a workflow document following a strict anatomy (see [CONTRIBUTING.md](CONTRIBUTING.md)). The content IS the product — there is no application code, no build step. Markdown quality, anatomy compliance, and cross-link integrity are the whole game.

## Repo structure

```
skills/<category>/<skill-name>/SKILL.md    # the product
skills/using-these-skills/SKILL.md         # the router meta-skill (top-level, no category)
docs/                                      # install guides per tool + comparison.md
.github/                                   # issue/PR templates
CONTRIBUTING.md                            # the anatomy spec — read before touching any SKILL.md
```

## Conventions you must follow

1. **Anatomy compliance.** Every SKILL.md follows CONTRIBUTING.md's section spec: frontmatter (name + description with triggers AND NOT-for exclusions), Overview, When to Use (+ When NOT), Prerequisites, The Workflow, Common Rationalizations (two-column table), Red Flags, Verification (evidence-based checkboxes), Example, Related skills. Don't invent new sections; don't drop existing ones.
2. **`name` matches the folder**, kebab-case, always.
3. **Cross-links must resolve.** Backtick skill references in Related skills sections must name skills that exist. Validate after any rename/removal:
   ```bash
   # every SKILL.md name/folder/sections check — run from repo root
   python - <<'EOF'
   import os, re, yaml
   for cat in os.listdir("skills"):
       p = os.path.join("skills", cat)
       for root, _, files in os.walk(p):
           if "SKILL.md" in files:
               t = open(os.path.join(root, "SKILL.md"), encoding="utf-8").read()
               fm = yaml.safe_load(re.match(r"^---\n(.*?)\n---", t, re.S).group(1))
               assert fm["name"] == os.path.basename(root), root
   print("names ok")
   EOF
   ```
4. **One skill = one job.** A skill that forks into alternative workflows is two skills — and the pre-flight checklist prefers extending an existing skill over adding a near-duplicate.
5. **Descriptions are routing signals:** triggers + exclusions, no workflow summaries, under ~90 words.
6. **Don't create empty `scripts/`/`templates/`/`reference/` dirs.** Only where a skill genuinely needs the asset.
7. **Commit style:** conventional commits, scoped by category (`feat(security): ...`, `refactor(testing-qa): ...`, `docs: ...`). No AI co-author trailers.
8. **README's category tree and counts** must stay accurate when skills are added/removed — update it in the same PR.

## What NOT to do

- Don't summarize or condense skills "for brevity" — the specificity is the value.
- Don't soften Rationalizations tables into generic advice; the excuses must be real and the rebuttals must survive scrutiny.
- Don't convert evidence-based Verification items into vague ones ("tests pass" without the artifact).
- Don't edit `natural-prose-editing` into a detector-evasion tool — its scope exclusion is deliberate and load-bearing.
- Don't reorganize categories without an issue + maintainer sign-off first.

## Testing your changes

No build; validation is structural. Run the name/anatomy check above, confirm cross-links resolve, and render the markdown (tables and code fences intact). For new skills, self-review against the PR template's anatomy checklist before opening the PR.
