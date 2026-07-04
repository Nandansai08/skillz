---
name: api-design-rest
description: >
  Use when designing or reviewing a REST/HTTP API — resource naming, status
  codes, pagination, versioning, error shapes, idempotency. Triggers:
  "design an endpoint", "REST API for", "what status code should", "how
  should I paginate", "version this API", "review my API design". NOT for
  GraphQL or RPC-style internal APIs (different trade-offs) and NOT for
  writing consumer documentation of a finished API (see api-documentation).
---

# REST API Design

## Overview

An API is a contract that outlives its implementation — naming, status codes, pagination, and versioning decisions made in an afternoon get consumed by clients for years. Designing against the conventions clients already expect is most of the job.

## When to Use

- Designing new endpoints or reviewing an API spec/PR.
- Deciding pagination, versioning, or error format for a service.

**When NOT to use:**
- GraphQL or internal RPC — different trade-off structure.
- Documenting an existing contract for consumers — `api-documentation`.

## Prerequisites

- List of the operations clients actually need (not the DB schema — the client's jobs).

## The Workflow

1. **Model resources as nouns, actions as HTTP verbs.**
   - Collections plural: `/orders`, `/orders/{id}`, `/orders/{id}/items`.
   - Nesting max one level deep; beyond that use filters: `/items?order_id=123` not `/customers/1/orders/123/items/9`.
   - Genuinely non-CRUD actions get a verb sub-resource as last resort: `POST /orders/{id}/cancel`. Don't contort "cancel" into a PATCH on status if the action has side effects (refunds, emails).

2. **Pick status codes from the short list.** 200 (read/update OK), 201 + `Location` header (created), 204 (no body: delete), 400 (malformed request), 401 (who are you), 403 (you can't), 404 (not found — also use for existence-leak-sensitive 403s), 409 (conflict/duplicate), 422 (well-formed but semantically invalid), 429 (rate limited, include `Retry-After`). Anything else needs a written justification.

3. **One error shape, everywhere.**
   ```json
   { "error": { "code": "ORDER_ALREADY_SHIPPED", "message": "Order 123 shipped on 2026-06-01 and cannot be cancelled.", "details": [] } }
   ```
   `code` is machine-readable and stable (clients branch on it); `message` is human-readable and changeable. Never make clients parse messages.

4. **Paginate every collection from day one.** Default: cursor-based (`?cursor=...&limit=50`, response carries `next_cursor`). Offset pagination is acceptable only for small, admin-facing, rarely-changing data — it skips/duplicates rows under concurrent writes and dies on deep pages. Cap `limit` server-side (e.g. 200).

5. **Make writes idempotent where retries are plausible.** `PUT` and `DELETE` are idempotent by definition; for `POST` that creates, accept an `Idempotency-Key` header and dedupe. Payment/ordering APIs: non-negotiable.

6. **Version in the URL path (`/v1/`), and mostly don't bump it.** Additive changes (new fields, new endpoints) are non-breaking — clients must ignore unknown fields, state this in the docs. Reserve v2 for genuine contract breaks, and plan the v1 deprecation window when you ship it, not after. Warning: timestamps RFC 3339 UTC always, and booleans that could grow a third state ship as string enums now — both are one-way doors.

7. **Write the spec (OpenAPI) before the handler.** A reviewable contract catches naming and shape problems while they cost nothing. Include at least one example request/response per endpoint, including one error.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "We'll add pagination when the table gets big" | By then clients depend on receiving everything, and adding pagination IS the breaking change. Day-one pagination costs one parameter. |
| "200 with success:false is simpler for our frontend" | It breaks every HTTP client's error handling, retry logic, caching, and monitoring — including your own future tooling. The status code IS the protocol. |
| "The DB schema is basically the API anyway" | Exposing `updated_ts`, join tables, and internal enums makes every schema migration a breaking change. The API is the client's contract, not the database's mirror. |
| "Idempotency keys are overkill for our volume" | Volume is irrelevant — one timeout-retry on a charge endpoint is one double-charge. The question is whether retries are plausible, and for POSTs they always are. |
| "We'll spec it in OpenAPI after the endpoints stabilize" | The spec-after-code sequence means the review happens after the shapes shipped — when fixing a name costs a version bump instead of a comment. |
| "Versioning can wait until we have external users" | Internal consumers calcify faster than external ones. The `/v1/` prefix costs four characters now and a migration project later. |

## Red Flags

- A collection endpoint returning an unbounded array.
- Error responses whose shape varies by endpoint, or clients doing string-matching on messages.
- `GET` endpoints with side effects; `POST` used for everything.
- Numeric booleans, local-time timestamps, or floats for money in the wire format.
- New endpoint merged with no OpenAPI entry and no error example.
- A "temporary" internal endpoint with no version prefix accumulating consumers.

## Verification

- [ ] Spec exists before implementation — OpenAPI diff linked in the PR.
- [ ] Every endpoint has ≥1 example request/response AND ≥1 error example in the spec.
- [ ] Every collection paginated with a server-side limit cap — shown in spec.
- [ ] Error shape uniform across all endpoints — spot-check three endpoints, attach outputs.
- [ ] Retry-plausible writes accept idempotency keys — demonstrated with a duplicate-request test.
- [ ] Status codes drawn from the step-2 list; exceptions justified in writing.

## Example

Requirement: clients cancel orders. Design produced: `POST /v1/orders/{id}/cancel` (side-effectful action, not PATCH), returns 200 with the updated order, 404 unknown order, 409 `ORDER_ALREADY_SHIPPED` if uncancellable, accepts `Idempotency-Key` so a retried cancel doesn't double-refund. Spec'd in OpenAPI with the 409 example; review caught that the mobile client also needed a cancellation reason — added as optional body field, non-breaking.

## Related skills

- `api-documentation` — documenting the finished contract for consumers.
- `error-handling-strategy` — the server-side half of the error shape decision.
- `architecture-decision-record` — recording the versioning/pagination choices and why.
