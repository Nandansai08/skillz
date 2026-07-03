---
name: monorepo-navigation
description: >
  Use when working in a large or unfamiliar repository and you need to find
  code, owners, or the blast radius of a change fast — search strategy,
  ownership files, build-graph queries. Triggers: "where is X defined",
  "who owns this code", "find all usages", "what depends on this package",
  "navigate this huge repo".
---

# Monorepo Navigation

## When to use this skill
- First week in a big repo, or any task starting with "find where...".
- Estimating blast radius before changing a shared package.
- NOT for understanding what found code *does* — see `legacy-code-first-contact` once you've located it.

## Prerequisites
- `ripgrep` (rg) or equivalent; repo cloned with full history (`--depth 1` clones break `git log` archaeology).

## Workflow

1. **Search definitions, not mentions.** Language-shaped patterns cut 500 hits to 3:
   ```bash
   rg "def process_payment|class PaymentProcessor" -t py
   rg "func ProcessPayment" -t go
   rg "(function|const) processPayment" -t ts
   ```
   Symbol indexes beat grep when available: IDE go-to-definition, `ctags`, Sourcegraph.

2. **Route by convention before searching.** Big repos are predictable: `services/<name>/`, `libs/`, `proto/` or `api/` for contracts, `cmd/` for entry points. Two minutes reading top-level directory names and the root README saves twenty of grepping.

3. **Find owners via metadata, then history.**
   - `CODEOWNERS` (root or `.github/`) — map path → team.
   - `git log --format='%an' -- path/to/dir | sort | uniq -c | sort -rn | head` — who actually touches it.
   - `git blame -w -C <file>` on the specific lines (`-w -C` skips whitespace/moves so you get the real author, not the reformatter).

4. **Measure blast radius with the build graph, not grep.** Grep finds textual mentions; the build graph finds real dependents:
   - Bazel: `bazel query 'rdeps(//..., //libs/auth:auth)'`
   - pnpm/turbo: `pnpm ls -r --filter '...@org/auth'` / `turbo run build --filter=...[HEAD^]`
   - Go: `go list -deps ./... | grep <module>` reversed via `grep -rl` on go.mod files.
   Fallback where no graph exists: grep the import statement form, not the bare name.

5. **Use history as a map.** "Where does feature X live?" → find one commit that touched it: `git log --oneline --grep="feature X" -i`, then `git show --stat <sha>` lists every file involved — instant feature map.

6. **Trace runtime paths from the entry point.** Route tables, DI wiring, and service mains are the trunk; grep the URL path or CLI command string to find the handler, then walk inward. Don't start from a leaf utility and try to walk outward.

7. **Cache what you learn.** Drop a `NOTES.md` (or team wiki entry) with the path map you built: entry points, ownership, "X actually lives in Y." Next person (often future you) starts at step 6 instead of step 1.

## Common pitfalls
- Grepping a generated tree: hits in `dist/`, `vendor/`, `node_modules/`, generated protos swamp the signal. Respect `.gitignore` (`rg` does by default) and add `-g '!*_pb2.py'`-style excludes for checked-in generated code.
- Trusting grep for blast radius in dynamic languages — reflection, string-built imports, and DI containers hide dependents. Cross-check with the build graph and runtime traces.
- Believing CODEOWNERS. It rots; the `git log` frequency check tells you who to actually ask.
- Deep-linking your mental model to paths. Monorepos reorganize; anchor on package/target names.

## Example
Task: change the signature of `libs/currency/convert()`. `rg "from libs.currency import"` shows 14 files; `bazel query 'rdeps(//..., //libs/currency)'` shows **31 targets** — grep missed re-exports through two facade packages. CODEOWNERS says team-payments; `git log` frequency says the active maintainer moved teams — actual contact found in one message instead of a dead channel. Change scoped to 31 targets, staged behind a new function name + deprecation instead of an in-place break.

## Related skills
- `legacy-code-first-contact` — what to do after you've found the code.
- `code-review-checklist` — blast-radius thinking applied to reviewing others' changes.
- `refactor-safely` — executing the cross-cutting change you just scoped.
