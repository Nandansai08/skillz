---
name: error-handling-strategy
description: >
  Use when deciding how code should handle failures — throw vs return vs
  log, where to catch, what to retry, auditing for swallowed errors.
  Triggers: "how should I handle this error", "try/catch here?", "should
  this throw", "error handling for this module", "silent failure", "empty
  catch block". NOT for designing HTTP error response shapes (see
  api-design-rest) and NOT for retry timing/backoff tuning (see
  retry-and-backoff-strategy).
---

# Error Handling Strategy

## Overview

Most production mysteries are swallowed errors: a catch block that logged-and-continued, a fallback that silently served defaults for six hours. Classifying failures into expected-vs-bug and catching only at boundaries makes failure paths as deliberate as success paths.

## When to Use

- Writing or reviewing failure paths: exceptions, error returns, fallbacks.
- Auditing a module for swallowed errors.

**When NOT to use:**
- HTTP wire-format design — `api-design-rest` step 3; this skill is the code behind it.
- Retry counts and backoff curves — `retry-and-backoff-strategy`; here the decision is only *whether* a call qualifies for retry.

## Prerequisites

- Know which failures are *expected* (user typo, missing record, timeout) vs *bugs* (violated invariant, impossible state). The whole strategy hinges on this split.

## The Workflow

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
   Wrap with cause preserved (`from e`, `%w`, `cause`). Log-and-rethrow at every layer is the anti-pattern — it produces five stack traces for one failure; log only at the boundary that handles it. Warning: error types need to carry the fields callers branch on — `PaymentError` without the decline code forces message-parsing downstream.

5. **Make fallbacks loud and bounded.** A fallback (cached value, default, degraded mode) needs: a log at WARN with the cause, a metric/counter, and a criterion for when it's unacceptable (e.g., serving stale prices OK for 5 min, not 5 hours). A silent fallback is a bug you've pre-authorized.

6. **Retry only idempotent operations, only on transient errors** (timeout, 503, connection reset — not 400, not auth failures). Tuning lives in `retry-and-backoff-strategy`; the decision here is just *whether* this call qualifies. A timeout on a non-idempotent write doesn't mean it didn't happen — the double-charge classic.

7. **Audit pass for swallowing.** Grep the diff/module for `except: pass`, `catch {}`, `catch (e) { console.log(e) }` (log-as-handling), and broad catches (`except Exception`) not at a boundary. Every hit either gets justified in a comment or fixed.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "I logged the error — it's handled" | Logging is not handling; the code after the catch now runs with broken assumptions. Handled means recovered, translated, or deliberately propagated. |
| "Broad catch is safer — the service stays up" | It catches the bug bin too, converting crashes into wrong answers served with a 200. A crash is diagnosable; corruption is archival. |
| "The fallback default keeps users unblocked" | Unmetered, it also keeps them unblocked during a total outage — serving defaults to 100% of traffic while dashboards stay green. Loud and bounded (step 5) or it's a pre-authorized incident. |
| "We'll clean up the empty catches later" | Each one is a landmine with no location record — later means "when one detonates in production and can't be traced." The audit grep is minutes. |
| "This can never fail here" | Then let it crash loudly when the impossible happens — that's the fail-fast bin working. Catching the impossible into a fallback is how impossible states propagate. |
| "Adding error types is bureaucracy — strings are fine" | String-matched errors are an API whose contract is a typo away from breaking. The fields callers branch on belong in the type. |

## Red Flags

- `except: pass` / `catch {}` anywhere outside a justified, commented boundary.
- The same failure logged at five stack levels — five traces, one incident, zero clarity.
- A fallback path with no metric — nobody can say how often it fires.
- Catch blocks inside pure logic functions (boundaries belong at edges).
- Retry decorators on non-idempotent writes.
- Post-incident finding: "the error was swallowed at..." — this skill's audit was never run.

## Verification

- [ ] Every failure point in the changed code binned (expected-recoverable / propagate / bug) — reviewable in the PR.
- [ ] Every catch block does one of: recover, wrap-and-rethrow with cause, translate at boundary — no exceptions unexplained.
- [ ] Fallbacks have log + metric + acceptability bound — metric name linked.
- [ ] Swallow-grep run on the module; hits fixed or comment-justified — grep output in PR.
- [ ] One failure-path test per expected-error bin (the error propagates/translates as designed) — test names listed.

## Example

Reviewing a sync job: `except Exception: logger.info("sync failed"); return []`. Classified: network errors = expected/transient → retry ×3 with backoff, then raise `SyncUnavailable`; malformed upstream payload = expected/not recoverable → raise with sample of payload; anything else = bug → propagate. Boundary (the cron wrapper) catches `SyncUnavailable` → alert-after-3-consecutive; returns nonzero on bugs so the scheduler pages. The `return []` silently feeding empty data downstream — which had been masking a week-long outage — is gone.

## Related skills

- `retry-and-backoff-strategy` — tuning the retries authorized in step 6.
- `alerting-design` — making step 5's fallback metrics page someone.
- `api-design-rest` — the wire-format shape these errors translate into at HTTP boundaries.
