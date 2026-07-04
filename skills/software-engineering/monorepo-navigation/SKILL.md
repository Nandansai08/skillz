---
name: monorepo-navigation
description: >
  Use when working in a large or unfamiliar repository and you need to find
  code, owners, or the blast radius of a change fast — search strategy,
  ownership signals, build-graph queries. Triggers: "where is X defined",
  "who owns this code", "find all usages", "what depends on this package",
  "navigate this huge repo". NOT for understanding what found code does
  (see legacy-code-first-contact once located).
---

# Monorepo Navigation

## Overview

Large repos reward strategy over stamina: definition-shaped searches, build-graph queries, and git archaeology answer in minutes what grep-and-scroll answers in hours — and blast-radius questions that grep answers *wrongly*.

## When to Use

- First week in a big repo, or any task starting with "find where...".
- Estimating blast radius before changing a shared package.

**When NOT to use:**
- Understanding located code's behavior — `legacy-code-first-contact`.

## Prerequisites

- `ripgrep` (rg) or equivalent; repo cloned with full history (`--depth 1` clones break `git log` archaeology).

## The Workflow

1. **Search definitions, not mentions.** Language-shaped patterns cut 500 hits to 3:
   ```bash
   rg "def process_payment|class PaymentProcessor" -t py
   rg "func ProcessPayment" -t go
   rg "(function|const) processPayment" -t ts
   ```
   Symbol indexes beat grep when available: IDE go-to-definition, `ctags`, Sourcegraph. Warning: exclude generated trees — hits in `dist/`, `vendor/`, checked-in protos swamp the signal (`rg` respects `.gitignore` by default; add `-g '!*_pb2.py'`-style excludes for committed generated code).

2. **Route by convention before searching.** Big repos are predictable: `services/<name>/`, `libs/`, `proto/` or `api/` for contracts, `cmd/` for entry points. Two minutes reading top-level directory names and the root README saves twenty of grepping.

3. **Find owners via metadata, then history.**
   - `CODEOWNERS` (root or `.github/`) — map path → team. It rots; verify with:
   - `git log --format='%an' -- path/to/dir | sort | uniq -c | sort -rn | head` — who actually touches it.
   - `git blame -w -C <file>` on the specific lines (`-w -C` skips whitespace/moves so you get the real author, not the reformatter).

4. **Measure blast radius with the build graph, not grep.** Grep finds textual mentions; the build graph finds real dependents:
   - Bazel: `bazel query 'rdeps(//..., //libs/auth:auth)'`
   - pnpm/turbo: `pnpm ls -r --filter '...@org/auth'` / `turbo run build --filter=...[HEAD^]`
   - Go: `go list -deps ./... | grep <module>` reversed via `grep -rl` on go.mod files.
   Fallback where no graph exists: grep the import statement form, not the bare name. In dynamic languages, cross-check both — reflection, string-built imports, and DI containers hide dependents from both tools individually.

5. **Use history as a map.** "Where does feature X live?" → find one commit that touched it: `git log --oneline --grep="feature X" -i`, then `git show --stat <sha>` lists every file involved — instant feature map.

6. **Trace runtime paths from the entry point.** Route tables, DI wiring, and service mains are the trunk; grep the URL path or CLI command string to find the handler, then walk inward. Don't start from a leaf utility and try to walk outward.

7. **Cache what you learn.** Drop a `NOTES.md` (or team wiki entry) with the path map you built: entry points, ownership, "X actually lives in Y." Next person (often future you) starts at step 6 instead of step 1.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "Grep found 14 usages — that's the blast radius" | Grep found 14 textual mentions. Re-exports, facades, and DI hide the rest; the build graph in the example below found 31. Textual search UNDERCOUNTS dependents structurally. |
| "CODEOWNERS says team-payments owns it, I'll ask them" | CODEOWNERS is where ownership was, not where it is. The git-log frequency check costs ten seconds and finds the human who actually answers. |
| "I'll just read the codebase top-down to understand it first" | A monorepo doesn't have a top. Convention-routing plus entry-point tracing gets you to the relevant 2% without the tour of the other 98%. |
| "No time to write notes — I'll remember the layout" | You won't, and neither will the next person, who re-derives your two hours from scratch. The NOTES.md is five minutes and compounds. |
| "The IDE index is down/slow, I'll pattern-match by filename" | Filenames lie in old repos (the `utils.py` with the payment logic). Definition-shaped rg patterns work everywhere the index doesn't. |

## Red Flags

- A shared-package change PR'd with no dependents query in evidence.
- "Who owns this?" answered from CODEOWNERS alone for anything that matters.
- Search sessions dominated by hits in `vendor/`, `dist/`, or generated files.
- Deep-linking documentation/mental models to file paths in a repo that reorganizes (anchor on package/target names).
- The same "where does X live?" question asked in channel twice in a month — nobody cached the answer.

## Verification

- [ ] For blast-radius tasks: build-graph (or import-form grep) dependents list attached to the change — count stated.
- [ ] Owner contact verified via git-log frequency, not metadata alone.
- [ ] Search patterns excluded generated/vendored trees (visible in the recorded commands).
- [ ] For onboarding/mapping tasks: NOTES.md (or wiki entry) produced with entry points and ownership map — linked.

## Example

Task: change the signature of `libs/currency/convert()`. `rg "from libs.currency import"` shows 14 files; `bazel query 'rdeps(//..., //libs/currency)'` shows **31 targets** — grep missed re-exports through two facade packages. CODEOWNERS says team-payments; `git log` frequency says the active maintainer moved teams — actual contact found in one message instead of a dead channel. Change scoped to 31 targets, staged behind a new function name + deprecation instead of an in-place break.

## Related skills

- `legacy-code-first-contact` — what to do after you've found the code.
- `code-review-checklist` — blast-radius thinking applied to reviewing others' changes.
- `refactor-safely` — executing the cross-cutting change you just scoped.
