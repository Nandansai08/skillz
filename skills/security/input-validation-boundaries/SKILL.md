---
name: input-validation-boundaries
description: >
  Use when deciding where and how to validate untrusted input — trust
  boundaries, allowlists vs denylists, canonicalization order, validation
  vs output encoding. Triggers: "where should I validate this", "sanitize
  user input", "validation layer", "is this input safe", "allowlist or
  blocklist", "canonicalization". NOT for hunting existing injection bugs
  (see secure-code-review) — this skill is where the fixes should live.
---

# Input Validation Boundaries

## Overview

Validate once at the boundary into a typed structure, and let the type system carry the guarantee inward; make data safe for each interpreter at its sink. Teams that blur these two layers — "sanitizing" at input to protect sinks — reliably get both wrong.

## When to Use

- Designing the validation story for a service or reviewing scattered ad-hoc checks.
- Handling any input crossing a trust boundary: user forms, API bodies, webhooks, file uploads, queues fed by other teams.

**When NOT to use:**
- Finding the injection bugs in current code — `secure-code-review`.

## Prerequisites

- A picture of the system's trust boundaries (`threat-modeling` step 1 produces exactly this).

## The Workflow

1. **Validate at the boundary, once, into a typed structure.** The pattern: raw input → parse/validate at the edge → a typed object (pydantic/zod/serde struct) that the rest of the code trusts. "Parse, don't validate": after the boundary, the type system carries the guarantee, and interior code stops re-checking (or worse, half-checking). Interior functions receiving raw dicts/strings from "somewhere" is the architecture smell this skill exists to fix.

2. **Know what validation can and cannot do.** Validation enforces *your business domain* (plausible email, amount 0–10000, status in enum). It does NOT make data safe for a specific interpreter — that's the sink's job: parameterized queries for SQL, output encoding for HTML, argument arrays for shells (`secure-code-review` step 2). `O'Brien` is valid input AND dangerous SQL; both handled, at different layers.

3. **Allowlist by shape, don't denylist by badness.** Define what valid looks like — type, length bounds, charset/pattern, range, enum membership — and reject everything else. Denylists (`strip <script>`, `block ../`) lose to encodings you didn't think of, forever. Practical floor for every string field: max length (unbounded strings are DoS + storage abuse) and encoding validity (reject broken UTF-8, reject null bytes).

4. **Canonicalize before validating, then never re-transform.** Decode URL-encoding, resolve unicode normalization (NFC), resolve paths — *then* check the result. Checking `../` before decoding misses `%2e%2e%2f`. And validate-then-use must not re-transform: a path checked then later joined again reopens the hole.

5. **Validate the semantic layer too, per input class:**
   - **Files:** verify content type by magic bytes not extension/header, cap size *before* buffering, re-encode images (strips polyglots + EXIF), never serve uploads from your origin without forcing download or a sandboxed domain.
   - **Webhooks/callbacks:** signature verification IS the validation; timestamp window against replay.
   - **URLs you'll fetch:** scheme+host allowlist (SSRF — `secure-code-review` step 5).
   - **Numbers:** range AND type (JSON `1e309`, `-0`, numeric strings); money as integer cents or Decimal, never float.
   - **IDs:** format-validate, but authorization is a separate check (`authn-authz-design`) — a well-formed ID is still someone else's.

6. **Reject, don't repair.** Auto-fixing invalid input (truncating, stripping, coercing) creates disagreement between what was sent, stored, and assumed downstream — and the repaired value may pass a check the original wouldn't. Reject with a specific error. Exception: normalization that's part of the *domain* (trimming email whitespace, lowercasing) — defined in the schema, once.

7. **Centralize the schemas and test the boundary.** One schema definition per input shape, shared/generated (OpenAPI/JSON Schema/zod) so docs, validation, and clients agree. Test with `edge-case-enumeration`'s string/number/collection lists — boundary tests are the cheapest security tests you own.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "We sanitize on input — HTML-escaping everything as it arrives" | The data also goes to SQL, logs, CSV, and email — each sink needs its own encoding, and the pre-mangled data breaks the non-HTML ones. Store truth; encode at output. |
| "The frontend validates all this already" | Client validation is UX. The API repeats everything or enforces nothing — bots, curl, and the mobile app's old version don't run your JavaScript. |
| "It comes from our own internal queue — already validated" | The producer's validation drifts from the consumer's assumptions, and the queue outlives both teams' memory. Internal boundaries are boundaries. |
| "Our regex blocks ../ and <script> — covered" | Denylists lose to `%2e%2e%2f`, unicode confusables, and next year's encoding. Allowlist the valid shape; the attacker enumerates badness better than you. |
| "Auto-correcting bad input is friendlier than rejecting" | The 'friendly' repair created free products in the example below — repaired values pass checks the original wouldn't, and three systems now disagree about what was sent. |
| "A max length on every string is bureaucratic" | The 60MB JSON body that OOMs the pod disagrees. Length caps are the cheapest DoS control in existence — one schema attribute. |

## Red Flags

- Validation logic scattered across three call-stack layers, each assuming another did it.
- Interior business functions typed as `dict`/`any` for boundary-crossing data.
- Sanitizer functions named `clean_*` doing HTML-escaping for storage.
- File uploads trusted by extension or Content-Type header.
- Webhook handlers with no signature verification.
- Repair-on-invalid behavior anywhere in the parse path.
- String fields with no length bounds in the schema.

## Verification

- [ ] One boundary per input shape: the typed-parse location named; interior code takes typed objects — architecture visible in the diff.
- [ ] Schemas centralized/shared — the single source linked; client/docs generated from it.
- [ ] Allowlist shapes with length + encoding floors on every string — schema shown.
- [ ] Canonicalization order verified: decode/normalize precedes checks — test with `%2e%2e%2f`-class inputs passing/failing correctly.
- [ ] Reject-not-repair: invalid inputs produce specific errors, zero silent fixes — rejection tests linked.
- [ ] Boundary test battery from edge-case-enumeration run — results linked.

## Example

Review of an import endpoint accepting CSV of products. Before: handler read raw rows, ad-hoc checks scattered in three functions, one path repaired bad prices by defaulting to 0 (silently created free products — real incident). After: multipart size cap 10MB at the gateway; magic-byte check (rejects the .xlsx people keep uploading, with a specific error); rows parsed through one pydantic schema — price as Decimal 0.01–100000, name ≤ 200 chars valid-UTF-8, category from enum; any invalid row → line-numbered rejection report, nothing repaired; SKU used in a path for the product image → canonicalized and prefix-checked at the storage layer (the sink), not the parser. The free-products class of bug is now structurally impossible.

## Related skills

- `secure-code-review` — finding where current code violates this design.
- `threat-modeling` — locating the boundaries this skill fortifies.
- `edge-case-enumeration` — the test inputs for step 7.
- `error-handling-strategy` — what rejection looks like in code.
