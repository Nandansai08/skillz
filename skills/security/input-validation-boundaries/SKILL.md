---
name: input-validation-boundaries
description: >
  Use when deciding where and how to validate untrusted input — trust
  boundaries, allowlists vs denylists, canonicalization order, validation
  vs output encoding. Triggers: "where should I validate this", "sanitize
  user input", "validation layer", "is this input safe", "allowlist or
  blocklist", "canonicalization".
---

# Input Validation Boundaries

## When to use this skill
- Designing the validation story for a service or reviewing scattered ad-hoc checks.
- Handling any input that crosses a trust boundary: user forms, API bodies, webhooks, file uploads, message queues fed by other teams.
- NOT for hunting existing injection bugs — that's `secure-code-review`; this skill is where the fixes should live.

## Prerequisites
- A picture of the system's trust boundaries (`threat-modeling` step 1 produces exactly this).

## Workflow

1. **Validate at the boundary, once, into a typed structure.** The pattern: raw input → parse/validate at the edge → a typed object (pydantic/zod/serde struct) that the rest of the code trusts. "Parse, don't validate": after the boundary, the type system carries the guarantee, and interior code stops re-checking (or worse, half-checking). Interior functions receiving raw dicts/strings from "somewhere" is the architecture smell this skill exists to fix.

2. **Know what validation can and cannot do.** Validation enforces *your business domain* (this is a plausible email, amount is 0–10000, status is one of three values). It does NOT make data safe for a specific interpreter — that's the sink's job: parameterized queries for SQL, output encoding for HTML, argument arrays for shells (`secure-code-review` step 2). A name like `O'Brien` is valid input AND dangerous SQL; both handled, at different layers. Teams that "sanitize" at input to protect sinks get both layers wrong.

3. **Allowlist by shape, don't denylist by badness.** Define what valid looks like — type, length bounds, charset/pattern, range, enum membership — and reject everything else. Denylists (`strip <script>`, `block ../`) lose to encodings you didn't think of, forever. Practical floor for every string field: max length (unbounded strings are DoS + storage abuse) and encoding validity (reject broken UTF-8, reject null bytes).

4. **Canonicalize before validating, then never re-transform.** Decode URL-encoding, resolve unicode normalization (NFC), resolve paths — *then* check the result. Checking `../` before decoding misses `%2e%2e%2f`; checking after your framework double-decodes misses the other direction. And validate-then-use must not re-transform: a path checked then later joined again reopens the hole (TOCTOU for parsers).

5. **Validate the semantic layer too, per input class:**
   - **Files:** verify content type by magic bytes not extension/Content-Type header, cap size *before* buffering, re-encode images (strips polyglots + EXIF), never serve uploads from your origin without forcing download or a sandboxed domain.
   - **Webhooks/callbacks:** signature verification is the validation; timestamp window against replay.
   - **URLs you'll fetch:** scheme+host allowlist (SSRF — `secure-code-review` step 5).
   - **Numbers:** range AND type (JSON `1e309`, `-0`, strings-that-look-numeric); money as integer cents or Decimal, never float.
   - **IDs:** format-validate, but remember authorization is a separate check (`authn-authz-design`) — a well-formed ID is still someone else's.

6. **Reject, don't repair.** Auto-fixing invalid input (truncating, stripping chars, coercing) creates disagreement between what the user sent, what you stored, and what downstream systems think — and the repaired value may pass a check the original wouldn't. Reject with a specific error (`error-handling-strategy`'s typed errors; field-level messages per `api-design-rest` step 3). Exception: normalization that's part of the *domain* (trimming whitespace on emails, lowercasing) — define it in the schema, once.

7. **Centralize the schemas and test the boundary.** One schema definition per input shape, shared/generated (OpenAPI/JSON Schema/zod) so the docs, the validation, and the client agree. Test with `edge-case-enumeration`'s string/number/collection lists — boundary tests are the cheapest security tests you own.

## Common pitfalls
- Validation sprinkled through the call stack: three layers each half-checking, every layer assuming another one did it. One boundary, typed handoff.
- "Sanitizing" for HTML at input time — the data goes to SQL, logs, CSV, and email too; each sink needs its own encoding, and the pre-mangled data breaks the non-HTML ones. Store the true value; encode at output.
- Trusting internal queues/services as "already validated" — the producer's validation drifts from the consumer's assumptions, and the queue outlives both teams' memory. Internal boundaries are boundaries.
- Client-side validation treated as validation. It's UX; the API repeats everything.
- Regex email/URL validation elaborate enough to reject real addresses while proving nothing (send the confirmation email — that's the validation). Shape-check loosely, verify semantically.
- Length limits absent until a 60MB JSON body OOMs the pod. Body-size caps at the edge, field caps in the schema.

## Example
Review of an import endpoint accepting CSV of products. Before: handler read raw rows, ad-hoc checks scattered in three functions, one path repaired bad prices by defaulting to 0 (silently created free products — real incident). After: multipart size cap 10MB at the gateway; magic-byte check (rejects the .xlsx people keep uploading, with a specific error); rows parsed through one pydantic schema — price as Decimal 0.01–100000, name ≤ 200 chars valid-UTF-8, category from enum; any invalid row → line-numbered rejection report, nothing repaired; SKU used in a path for the product image → canonicalized and prefix-checked at the storage layer (the sink), not the parser. The free-products class of bug is now structurally impossible.

## Related skills
- `secure-code-review` — finding where current code violates this design.
- `threat-modeling` — locating the boundaries this skill fortifies.
- `edge-case-enumeration` — the test inputs for step 7.
- `error-handling-strategy` — what rejection looks like in code.
