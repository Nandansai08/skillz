---
name: api-documentation
description: >
  Use when documenting an API for consumers — endpoint reference with
  examples first, error catalogs, auth walkthroughs, keeping docs and
  implementation in sync. Triggers: "document this API", "write API
  docs", "API reference", "developers keep asking how to call this",
  "OpenAPI descriptions", "document these endpoints". NOT for designing
  the API itself (see api-design-rest) — good docs can't rescue a
  confusing contract, and the doc pass often reveals design bugs.
---

# API Documentation

## Overview

The metric is time-to-first-successful-call, and the two levers are a complete runnable auth walkthrough and an error catalog that tells integrators what to DO. Docs written without executing every example are confidently wrong somewhere — always.

## When to Use

- Publishing or overhauling reference docs for an HTTP API (internal or external).
- Support/integration questions reveal the docs aren't answering real questions.

**When NOT to use:**
- Designing the contract — `api-design-rest`; report design bugs the doc pass reveals, don't paper over them.

## Prerequisites

- A working API to document, with the spec (OpenAPI or equivalent) if one exists.
- Credentials for a sandbox — every example you write must be executed, not imagined.

## The Workflow

1. **Lead with the getting-to-first-call walkthrough, not the reference.** Page one: get credentials → make this exact curl request → see this exact response. Auth is where most first calls die, so show the *complete* working request with header placement, not a prose description of OAuth:
   ```bash
   curl https://api.example.com/v1/orders \
     -H "Authorization: Bearer sk_test_..."
   # → 200 {"data": [...], "next_cursor": "..."}
   ```

2. **Structure every endpoint's reference identically:** one-line purpose → request (method, path, params table: name, type, required?, constraints, default) → a *realistic* example request → example success response with field notes where non-obvious → the endpoint's specific errors. Uniformity is a feature: readers learn the shape once, then scan.

3. **Examples first, schemas second — and make examples realistic.** Developers copy examples and adapt; they read schemas only when stuck. Realistic means: plausible values (not `"string"`, `123`), optional-but-common fields present, formats taught implicitly (`"2026-07-03T14:00:00Z"` teaches the timestamp contract better than a paragraph).

4. **Document errors as a first-class catalog.** Integrators spend more time on error paths than happy paths. Every `code` listed with meaning, retryability, and what the caller should *do* ("`RATE_LIMITED`: retry after the `Retry-After` value; sustained 429s mean raise your tier"). Plus the cross-cutting behaviors nobody documents and everybody needs: rate limits and their headers, idempotency-key semantics, pagination contract (cursor lifetime!), webhook retry policy.

5. **Generate the skeleton, write the flesh.** Generate the mechanical layer from the spec (OpenAPI → reference pages) so params/types can't drift — but generated stubs are scaffolding. Human-written: purpose lines, examples, error guidance, "when to use this vs that endpoint." Docs that are 100% generated read as a schema dump and answer no real questions.

6. **Wire the drift alarms:** examples executed against the sandbox in CI (a failing docs-test on an API change is the system working — that change was breaking, `release-versioning` step 2); spec-vs-implementation contract checks; and a dated changelog for the API itself with migration notes.

7. **Close the loop from real friction:** every recurring support/integration question is a docs defect — answer it, patch the doc the same day, link it. Quarterly, skim docs-site searches/404s: what integrators search for and don't find is the roadmap.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "The OpenAPI spec IS the documentation" | A schema dump answers 'what fields exist,' never 'how do I take a payment.' The generated layer is scaffolding; the walkthrough, examples, and error guidance are the docs. |
| "Auth is standard OAuth — link to the RFC" | Auth is where most first calls die, and the RFC doesn't show YOUR header placement and YOUR token prefix. The complete runnable request is the single highest-value block in the docs. |
| "Placeholder examples are cleaner: field: string" | Placeholders can't be copy-pasted into a working call and teach no formats. Realistic sanitized payloads are what integrators actually build from. |
| "We document success; errors are self-explanatory" | The 409 the integrator WILL hit is where docs quality is decided — integrators live on error paths. The catalog with do-this guidance is the difference between docs and decoration. |
| "The spec says it's optional — no need to test the example" | The spec said optional; the server 400s without it. Every example executed against the sandbox (step 6 makes it permanent) or the docs are confidently wrong somewhere. |
| "Support can keep answering that question" | Twice-explained = missing doc. Each recurring ticket is a defect with a same-day patch; the loop (step 7) is what bends the ticket curve. |

## Red Flags

- First-call failures dominated by auth confusion.
- Examples that have never been executed.
- Error codes existing in code with no doc entry.
- Pagination/rate-limit/idempotency semantics undocumented (discovered by incident).
- Docs-test absent; breaking changes discovered by integrators.
- The same support question answered three times with no doc patch.

## Verification

- [ ] First-call walkthrough executed start-to-finish from the docs alone — by someone without prior setup; time recorded.
- [ ] Every endpoint follows the identical reference shape — spot-check three.
- [ ] Error catalog complete against the code's error-code enum — diff shown.
- [ ] All examples executed in CI against sandbox — job linked, green.
- [ ] Cross-cutting behaviors (rate limits, idempotency, pagination lifetimes, webhooks) each have a section.
- [ ] Last quarter's recurring support questions each traceable to a doc patch — links.

## Example

Payments API, external integrators, ~30 support tickets/month tagged "docs". Audit of a month's tickets: 11 were auth-header confusion, 8 undocumented error codes, 5 pagination-cursor expiry surprises. Rewrite: first-call walkthrough with complete curl + expected response (auth tickets → 2/month); error catalog generated from the code's error-code enum + hand-written "what to do" per code, including the 3 codes that existed in code but no doc; pagination contract documented with cursor TTL (24h) — which the doc pass revealed was *undefined in the implementation* (cursors lived until a deploy — a real design bug, ticketed and fixed). Examples wired to CI against sandbox; two months later a PR renaming a response field failed the docs test — caught as breaking before ship. Tickets: 30/month → 6/month.

## Related skills

- `api-design-rest` — the contract these docs describe; doc pain is design feedback.
- `docs-information-architecture` — reference vs how-to layering.
- `release-versioning` — the API changelog and breaking-change discipline.
- `readme-authoring` — the same show-the-output philosophy, repo-scale.
