---
name: api-documentation
description: >
  Use when documenting an API for consumers — endpoint reference with
  examples first, error catalogs, auth walkthroughs, keeping docs and
  implementation in sync. Triggers: "document this API", "write API docs",
  "API reference", "developers keep asking how to call this", "OpenAPI
  descriptions", "document these endpoints".
---

# API Documentation

## When to use this skill
- Publishing or overhauling reference docs for an HTTP API (internal or external).
- Support/integration questions reveal the docs aren't answering real questions.
- NOT for designing the API itself — that's `api-design-rest`; good docs can't rescue a confusing contract, and the doc pass often reveals design bugs (report them, don't paper over them).

## Prerequisites
- A working API to document, with the spec (OpenAPI or equivalent) if one exists.
- Credentials for a sandbox/test environment — every example you write must be executed, not imagined.

## Workflow

1. **Lead with the getting-to-first-call walkthrough, not the reference.** The killer metric for API docs is time-to-first-successful-call. Page one: get credentials → make this exact curl request → see this exact response. Auth is where most first calls die, so show the *complete* working request with header placement, not a prose description of OAuth:
   ```bash
   curl https://api.example.com/v1/orders \
     -H "Authorization: Bearer sk_test_..." 
   # → 200 {"data": [...], "next_cursor": "..."}
   ```

2. **Structure every endpoint's reference identically:** one-line purpose → the request (method, path, params in a table: name, type, required?, constraints, default) → a *realistic* example request → the example success response with field-by-field notes where non-obvious → the endpoint's specific errors. Uniformity is a feature: readers learn the shape once, then scan.

3. **Examples first, schemas second — and make examples realistic.** Developers copy examples and adapt; they read schemas only when stuck. Realistic means: plausible values (not `"string"`, `"foo"`, `123`), the optional-but-common fields present, and amounts/dates showing the format contract implicitly (`"2026-07-03T14:00:00Z"` teaches the timestamp format better than a paragraph). One example per meaningfully different call pattern, not one per endpoint.

4. **Document errors as a first-class catalog.** Integrators spend more time on error paths than happy paths. Per the API's error shape (`api-design-rest` step 3): every `code` listed with its meaning, whether it's retryable, and what the caller should *do* ("`RATE_LIMITED`: retry after the `Retry-After` header value; sustained 429s mean raise your tier"). Plus the cross-cutting behaviors nobody documents and everybody needs: rate limits and their headers, idempotency-key semantics, pagination contract (cursor lifetime!), timeout recommendations, webhook retry policy.

5. **Generate the skeleton, write the flesh.** Generate the mechanical layer from the spec (OpenAPI → reference pages) so params/types can't drift from the implementation — but the generated stubs are scaffolding, not docs. Human-written: the purpose lines, the examples, the error guidance, the "when to use this vs that endpoint" notes. Docs that are 100% generated read like a schema dump and answer no real questions.

6. **Wire the drift alarms:** examples executed against the sandbox in CI (a failing docs-test on an API change is the system working — that change was breaking, see `release-versioning` step 2); spec-vs-implementation contract checks; and a changelog for the API itself (dated, per-version, with migration notes for anything breaking).

7. **Close the loop from real friction:** every recurring support/integration question is a docs defect — answer it, then patch the doc the same day and link it. Quarterly, skim the search queries/404s on the docs site if available; what integrators search for and don't find is the roadmap.

## Common pitfalls
- Auth documented as prose ("authenticate using your API key") instead of a complete runnable request — the #1 source of failed first calls.
- Placeholder examples (`"field": "string"`) that teach nothing and can't be copy-pasted into a working call.
- Happy-path-only docs: the 200 documented lovingly, the 409 the integrator will definitely hit left as a mystery. Step 4's catalog is where doc quality is actually decided.
- Documenting from the spec without executing — the spec says the field is optional, the server 400s without it, and the docs are now confidently wrong. Execute everything (step 6 makes this permanent).
- The undocumented behaviors integrators discover by incident: pagination cursor expiry, rate-limit scope (per-key or per-endpoint?), webhook ordering guarantees. If support has explained it twice, it's a missing doc.
- Reference-only docs with no task guides: 40 perfectly-documented endpoints and no page answering "how do I take a payment" — the reference is the dictionary; integrators need the phrasebook too (`docs-information-architecture`'s how-to quadrant).

## Example
Payments API, external integrators, ~30 support tickets/month tagged "docs". Audit of a month's tickets: 11 were auth-header confusion, 8 were undocumented error codes, 5 were pagination-cursor expiry surprises. Rewrite: first-call walkthrough with complete curl + expected response (auth tickets → 2/month); error catalog generated from the actual error-code enum in the codebase + hand-written "what to do" per code, including the 3 codes that existed in code but no doc; pagination contract documented with cursor TTL (24h) — which the doc pass revealed was *undefined in the implementation* (cursors lived until a deploy — a real design bug, ticketed and fixed per the not-for-this-skill note). Examples wired to CI against sandbox; two months later a PR renaming a response field failed the docs test — caught as breaking before ship, exactly the drift alarm working. Tickets: 30/month → 6/month.

## Related skills
- `api-design-rest` — the contract these docs describe; doc pain is design feedback.
- `docs-information-architecture` — reference vs how-to layering.
- `release-versioning` — the API changelog and breaking-change discipline.
- `readme-authoring` — the same examples-with-output philosophy, repo-scale.
