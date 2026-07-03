---
name: error-handling-strategy
description: >
  Use when deciding how code should handle failures — throw vs return vs log,
  where to catch, what to retry, how to avoid swallowing errors. Triggers:
  "how should I handle this error", "try/catch here?", "should this throw",
  "error handling for this module", "silent failure", "empty catch block".
---

# Error Handling Strategy

## When to use this skill
- Writing or reviewing failure paths: exceptions, error returns, fallbacks.
- Auditing a module for swallowed errors.
- NOT for designing HTTP error responses — see `api-design-rest` step 3 for the wire format; this skill is the code behind it.

## Prerequisites
- Know which failures are *expected* (user typo, missing record, timeout) vs *bugs* (violated invariant, impossible state). The whole strategy hinges on this split.

## Workflow

1. **Classify every failure point into three bins.**
   - **Expected, recoverable** (not-found, validation, timeout): part of the function's contract. Return it as a value where the language supports it well (`Result`, `Option`, error return) or throw a *typed* exception the caller is expected to catch.
   - **Expected, not locally recoverable** (DB down): throw/propagate; recovery belongs to a boundary, not this function.
   - **Bug** (invariant broken): fail fast — assert/panic/throw unconditionally. Never catch these into a fallback; a fallback on a bug converts a crash into silent corruption.

2. **Catch only where you can do one of three things:** recover meaningfully, add context and rethrow, or translate at a boundary. A catch block that can't do any of these shouldn't exist — let it propagate.

3. **Put boundaries in explicit places.** Typical set: request handler (translate to HTTP error), job/queue consumer (retry or dead-letter), main/CLI (message + exit code), thread/task spawn points (crashed background work must surface, not vanish). Everything between boundaries propagates.

4. **Add context on the way up, once per layer.**
   ```python
   raise OrderLoadError(f"loading order {order_id} for invoice {invoice_id}") from e
   ```
   Wrap with cause preserved (`from e`, `%w`, `cause`). Log-and-rethrow at every layer is the anti-pattern — it produces five stack traces for one failure; log only at the boundary that handles it.

5. **Make fallbacks loud and bounded.** A fallback (cached value, default, degraded mode) needs: a log at WARN with the cause, a metric/counter, and a criterion for when it's unacceptable (e.g., serving stale prices OK for 5 min, not 5 hours). A silent fallback is a bug you've pre-authorized.

6. **Retry only idempotent operations, only on transient errors** (timeout, 503, connection reset — not 400, not auth failures), with capped attempts and backoff. See `retry-and-backoff-strategy` for tuning; here the decision is just *whether* this call qualifies.

7. **Audit pass for swallowing.** Grep the diff/module for `except: pass`, `catch {}`, `catch (e) { console.log(e) }` (log-as-handling), and broad catches (`except Exception`) not at a boundary. Every hit either gets justified in a comment or fixed.

## Common pitfalls
- Catch-log-continue as reflex. Logging is not handling; the code after the catch now runs with broken assumptions.
- Broad `except Exception` around a big block "to be safe" — it catches the bug bin too, turning crashes into wrong answers. Narrow the type or narrow the block.
- Fallback masking total outage: default-on-error made 100% of requests serve the default for six hours and no one noticed. That's what step 5's metric is for.
- Error types with no data. `PaymentError` without the decline code forces callers to parse messages — carry the fields callers branch on.
- Retrying non-idempotent writes on timeout: the timeout doesn't mean it didn't happen. Double-charge classic.

## Example
Reviewing a sync job: `except Exception: logger.info("sync failed"); return []`. Classified: network errors = expected/transient → retry ×3 with backoff, then raise `SyncUnavailable`; malformed upstream payload = expected/not recoverable → raise with sample of payload; anything else = bug → propagate. Boundary (the cron wrapper) catches `SyncUnavailable` → alert-after-3-consecutive; returns nonzero on bugs so the scheduler pages. The `return []` silently feeding empty data downstream — which had been masking a week-long outage — is gone.

## Related skills
- `retry-and-backoff-strategy` — tuning the retries authorized in step 6.
- `alerting-design` — making step 5's fallback metrics page someone.
- `api-design-rest` — the wire-format shape these errors translate into at HTTP boundaries.
