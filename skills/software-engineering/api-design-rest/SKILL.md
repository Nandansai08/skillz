---
name: api-design-rest
description: >
  Use when designing or reviewing a REST/HTTP API — resource naming, status
  codes, pagination, versioning, error shapes. Triggers: "design an endpoint",
  "REST API for", "what status code should", "how should I paginate",
  "version this API", "review my API design".
---

# REST API Design

## When to use this skill
- Designing new endpoints or reviewing an API spec/PR.
- Deciding pagination, versioning, or error format for a service.
- NOT for GraphQL or RPC-style internal APIs — different trade-offs.

## Prerequisites
- List of the operations clients actually need (not the DB schema — the client's jobs).

## Workflow

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

6. **Version in the URL path (`/v1/`), and mostly don't bump it.** Additive changes (new fields, new endpoints) are non-breaking — clients must ignore unknown fields, state this in the docs. Reserve v2 for genuine contract breaks, and plan the v1 deprecation window when you ship it, not after.

7. **Write the spec (OpenAPI) before the handler.** A reviewable contract catches naming and shape problems while they cost nothing. Include at least one example request/response per endpoint, including one error.

## Common pitfalls
- Mirroring the database: exposing `updated_ts`, join tables, internal enums. The API is a contract for clients; renaming a DB column shouldn't be a breaking change.
- 200 with `{"success": false}` in the body. Breaks every HTTP client's error handling, retry logic, and monitoring.
- Unbounded collections "for now." The day it hurts, clients already depend on getting everything, and adding pagination *is* the breaking change.
- Timestamps without timezone or in local time. RFC 3339 UTC (`2026-07-03T14:00:00Z`), always.
- Booleans that will become enums (`is_active` → later needs `suspended`). If a third state is conceivable, ship a string enum now.

## Example
Requirement: clients cancel orders. Design produced: `POST /v1/orders/{id}/cancel` (side-effectful action, not PATCH), returns 200 with the updated order, 404 unknown order, 409 `ORDER_ALREADY_SHIPPED` if uncancellable, accepts `Idempotency-Key` so a retried cancel doesn't double-refund. Spec'd in OpenAPI with the 409 example; review caught that the mobile client also needed a cancellation reason — added as optional body field, non-breaking.

## Related skills
- `api-documentation` — documenting the finished contract for consumers.
- `error-handling-strategy` — the server-side half of the error shape decision.
- `architecture-decision-record` — recording the versioning/pagination choices and why.
