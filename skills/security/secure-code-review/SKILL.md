---
name: secure-code-review
description: >
  Use when reviewing code specifically for security defects — injection,
  broken authz, secrets, unsafe deserialization — with concrete grep
  patterns per vulnerability class. Triggers: "security review this
  code", "is this code vulnerable", "check for injection", "audit this
  PR for security", "OWASP review". NOT for general code quality (see
  code-review-checklist — run this as its security step) and NOT for
  design-time threat analysis (see threat-modeling).
---

# Secure Code Review

## Overview

Security review is class-by-class pattern hunting ordered by real-world frequency — authz first, injection second — with each finding swept for its siblings across the codebase. One instance found and fixed is a patch; the class fixed structurally is security.

## When to Use

- A PR touches auth, user input, file handling, queries, or crypto — or a periodic security pass over a module.
- Following up a threat model's mitigations in the implementation.

**When NOT to use:**
- General correctness/style review — `code-review-checklist` (this is its step-3 deep pass).
- Pre-code design analysis — `threat-modeling`.

## Prerequisites

- The code plus its routing/entry-point context (a handler is only judgeable knowing what reaches it unauthenticated).
- Repo-wide grep access — security review is about the pattern's *other* instances too.

## The Workflow

Work the classes in order of real-world frequency. For each: grep the candidates, then read hits with the question "can attacker-controlled data reach this?"

1. **Broken authorization (the #1 web vulnerability class).** Every handler that takes an ID: is ownership/tenancy checked, or does possession of the ID suffice (IDOR)?
   ```
   rg "get_object|find_by_id|findOne|\.get\(.*_id" --type py -t ts
   ```
   Read each: does the query include `user_id=current_user` / tenant scope, or is it filtered only by the passed ID? Also: authz on *every* verb (the GET is checked, the DELETE forgot), mass-assignment (`**params`, `Object.assign(user, req.body)`) letting callers set `role=admin`.

2. **Injection — anywhere strings meet interpreters.**
   - SQL: `rg "f\"SELECT|\" \+ .*(WHERE|SELECT)|format\(.*SELECT|%s.*% \("` — any query built by concatenation. Parameterized queries or bust; `ORDER BY`/table-name interpolation needs allowlists (parameters can't cover identifiers).
   - Command: `rg "shell=True|os\.system|exec\(|child_process|Runtime\.getRuntime"` — argument arrays over shell strings; if shell is unavoidable, allowlist the input, don't escape it.
   - Path: `rg "open\(.*request|join\(.*(filename|params)|sendFile"` — resolve then verify prefix (`realpath` inside the base dir), reject `..` after decoding.
   - Template/eval: `render_template_string`, `eval(`, `new Function(` with any request-derived input.

3. **Secrets and sensitive-data handling.** `rg -i "(api_key|password|secret|token)\s*=\s*[\"']"` for hardcoded values (then `secrets-incident-response` if found in history); logging of tokens/PII (`rg "log.*\b(password|token|authorization|ssn)"`); sensitive fields in serializers/API responses that expose more than the UI shows.

4. **Unsafe deserialization and parsing:** `pickle.loads`, `yaml.load` (without `SafeLoader`), Java `ObjectInputStream`, `unserialize(` on anything request-derived — these are remote-code-execution, not "input validation issues." XML parsers with external entities enabled (XXE). `JSON.parse` is fine; it's the polymorphic/native deserializers that kill.

5. **SSRF and outbound fetches:** any `requests.get(url)` / `fetch(url)` where `url` derives from user input — can it reach `169.254.169.254` (cloud metadata) or internal services? Allowlist schemes+hosts; resolve-then-connect pinning for the serious cases.

6. **Crypto and randomness misuse:** `rg "random\.|Math\.random"` in token/reset-code generation (must be `secrets`/`crypto.randomBytes`); homemade password hashing (must be bcrypt/scrypt/argon2); `==` on MACs/tokens (timing — use constant-time compare); JWT: `alg` honored from the token, `none` accepted, or missing expiry validation.

7. **For every finding, sweep the codebase for siblings and fix the class.** One concatenated query found = grep the pattern everywhere; the fix should be structural where possible — a lint/semgrep rule banning the pattern turns today's finding into tomorrow's CI gate.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "Input is sanitized at the entry point, the sink is safe" | Sanitize-at-entry misses the second path to the sink — the queue consumer, the admin import, the internal caller. Review the sink's guarantees, not the entrance's intentions. |
| "The frontend prevents users from sending that" | The API is the product; the frontend is a suggestion. Authz and validation live server-side or they don't exist. |
| "Escaping the input is equivalent to parameterizing" | Escaping is the fallback that gets one edge case wrong per decade per codebase. Parameters are structurally immune; the choice isn't stylistic. |
| "Only our services call this endpoint" | Until the first SSRF hands an attacker an internal HTTP client. Internal-only is a network claim, not an authn design. |
| "Fixed the query the review flagged — done" | Vulnerabilities are habits: the author who concatenated once concatenated elsewhere. The sibling sweep (step 7) is half the review's value. |
| "That timing attack is theoretical — rate limiting covers it" | Maybe — but write THAT down as a risk decision, ranked against the unauthenticated IDOR you're spending equal time on. Severity ranking is part of the review, not an afterthought. |

## Red Flags

- Review comments all at one severity — no attacker-ranked prioritization.
- An IDOR fixed in one handler while sibling handlers keep the pattern.
- "Sanitization" functions doing HTML-escaping for SQL contexts.
- Findings without the grep evidence (which pattern, how many hits).
- Auth checks present on GET, absent on DELETE/PATCH of the same resource.
- No structural follow-up (lint rule/semgrep) on a repeated-pattern finding.

## Verification

- [ ] All seven classes walked; per-class grep patterns and hit counts recorded in the review.
- [ ] Every ID-taking handler in the diff checked for ownership/tenant scoping — noted per handler.
- [ ] Findings severity-ranked by attacker reachability × asset value.
- [ ] Sibling sweep run per finding — pattern, hits, and dispositions listed.
- [ ] Structural prevention shipped or ticketed for repeated patterns (semgrep/lint rule linked).
- [ ] Any hardcoded secret found routed through secrets-incident-response — link.

## Example

PR: "export orders to CSV." Review found: (1) `Order.objects.filter(id__in=ids)` — no user scoping; any authenticated user exports anyone's orders by ID list (IDOR, blocking). (2) Filename `f"export-{request.GET['name']}.csv"` written under `/exports` — path traversal via `name=../../cron/job` (blocking). (3) CSV cells starting with `=` unescaped — formula injection into victims' Excel (suggestion + sibling grep found the same in two other exporters). Fixes: queryset scoped to `request.user`, filename replaced by server-generated UUID, prefix-escaping in the shared exporter. Semgrep rule added banning unscoped `id__in` filters in handlers.

## Related skills

- `code-review-checklist` — the general review this slots into.
- `threat-modeling` — deciding which components deserve this depth.
- `input-validation-boundaries` — where the fixes belong architecturally.
- `dependency-vulnerability-audit` — the third-party half of code security.
