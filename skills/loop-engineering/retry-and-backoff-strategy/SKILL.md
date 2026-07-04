---
name: retry-and-backoff-strategy
description: >
  Use when deciding how to retry flaky operations — retry counts, backoff
  curves with jitter, circuit breakers, retry budgets — for API calls, tool
  calls, and agent actions alike. Triggers: "retry policy", "exponential
  backoff", "circuit breaker", "how many retries", "retry storm",
  "handle transient failures".
---

# Retry & Backoff Strategy

## When to use this skill
- Designing retry behavior for any flaky call: external APIs, LLM calls, agent tool invocations, network operations.
- A retry storm or thundering herd just happened, or a "we retry 3 times" policy exists that nobody can justify.
- NOT for deciding IF an error is retryable in the first place — `error-handling-strategy` step 6 classifies; this skill tunes what happens after the classification says yes.

## Prerequisites
- The operation's failure taxonomy: which errors are transient (timeout, 429, 503, connection reset), which are permanent (400, 401, 404, validation) — retrying permanents is pure waste and occasionally pure damage.
- Idempotency status of the operation (`data-pipeline-idempotency`, `api-design-rest` step 5) — non-idempotent + timeout = the double-charge; retries there need idempotency keys FIRST, tuning second.

## Workflow

1. **Gate retries on error class, and honor the server's own signals:** retry only the transient set; never 4xx-except-408/429; ALWAYS respect `Retry-After` headers and rate-limit metadata when present (the server is telling you the correct backoff — computing your own instead is both ruder and worse). Timeouts get special care: the operation may have SUCCEEDED (the response died, not the request) — safe to retry only with idempotency guaranteed.

2. **Default shape: exponential backoff with full jitter, and know why each part exists:**
   ```
   delay = random(0, min(cap, base × 2^attempt))    # full jitter
   # typical: base=1s, cap=30–60s, attempts=3–5
   ```
   Exponential because transient failures cluster (the struggling server needs relief that grows); the cap because unbounded exponentials produce 17-minute waits nobody wanted; **jitter because synchronized clients are the thundering herd** — a fleet retrying at identical deterministic delays re-spikes the recovering service at t+1s, t+2s, t+4s in unison (full jitter, the random(0, ceiling) form, decorrelates the herd and is the empirically strongest simple variant). Deterministic backoff in a fleet is a scheduled DDoS of your own dependency.

3. **Pick attempt counts from the caller's time budget, not folklore:** total worst-case time = Σ(timeouts + delays) — a user-facing request wrapping 5 attempts × 10s timeouts is a 50-second spinner (the correct user-facing answer is usually 1–2 fast retries then fail visibly, `empty-loading-error-states` step 4's retryable-error UI); background jobs afford patient curves (5+ attempts, minutes of cap, then dead-letter queue). Write the budget arithmetic in the config comment — "3 retries" without the derivation is cargo cult.

4. **Add the circuit breaker when callers are many or the dependency is shared:** per-dependency state machine — closed (normal) → open (failure rate over threshold: fail fast WITHOUT calling, giving the dependency genuine recovery room) → half-open (probe trickle; success closes, failure re-opens). Retries protect the individual call; the breaker protects the SYSTEM from everyone's retries compounding — at fleet scale, per-call retry×fleet = multiplied load exactly when the dependency can least afford it (the retry-storm mechanism `capacity-planning`'s pitfalls warn about). Libraries exist everywhere (resilience4j, Polly, pybreaker); the design decision is the thresholds and what "fail fast" returns (`error-handling-strategy`'s loud-fallback rules).

5. **Bound the aggregate with a retry budget:** cap retries as a fraction of total traffic (e.g., retries ≤ 10% of requests, token-bucket enforced) — under partial outage, per-call policies multiply load 3–5× fleet-wide even with jitter; the budget converts "every call retries independently" into "the system retries sustainably." Same concept for agent loops: the iteration's retry allowance draws from the run's total budget (`runaway-cost-guardrails`), so retry enthusiasm can't consume the task's whole allowance.

6. **For agent/LLM loops specifically, retry the right layer:** transport-level transients (429, 500, timeout) get the standard curve INSIDE the tool wrapper — invisible to the agent; semantic failures (bad output, failed validation) are NOT retries — they're new attempts that must differ (`tool-use-loop-debugging`'s perseveration boundary: transport retries repeat verbatim by design; semantic retries that repeat verbatim are the pathology). Conflating the layers produces agents that "retry" a reasoning failure five times at the API tier, or treat a rate-limit as evidence their approach failed.

7. **Instrument and verify under failure injection:** metrics per dependency — retry rate (rising = the dependency degrading BEFORE the outage), success-by-attempt-number (if attempt 3 never succeeds, attempts 2+ are pure waste: shrink the count), breaker state transitions (alert on open — `alerting-design` lanes); then chaos-test the policy (inject 50% failures in staging: does the herd stay decorrelated? does the breaker open? does the budget hold?) — a retry policy that's never seen injected failure is a hypothesis with production as its first test (`experiment-design` instincts, resilience edition).

## Common pitfalls
- Retrying everything: the 400 retried three times (same request, same rejection, tripled load, delayed honest failure) — the error-class gate (step 1) is the policy's foundation, not a refinement.
- Jitter-free exponential in a fleet: textbook curve, synchronized herd, the dependency re-killed at every power-of-two — jitter is not optional at fleet scale (step 2).
- Nested retry multiplication: the HTTP client retries ×3, the service layer ×3, the job runner ×3 — 27 actual attempts nobody designed; audit the STACK for hidden retry layers (the most common cause of "why did it call 27 times?").
- Retrying non-idempotent operations on timeout: the double-charge, the duplicate email, the twice-created resource — idempotency keys precede tuning (prerequisites).
- Retries masking chronic failure: 15% baseline failure rate invisible because retries absorb it — until load rises and the absorption capacity doesn't; the retry-rate metric (step 7) is the early-warning the masking steals.
- Breaker-free fleets: 400 instances each politely retrying ×3 = 1200 attempts against a dependency at its worst moment — individually correct, collectively lethal (steps 4–5).

## Example
Incident: payment provider had a 20-minute degradation; the checkout fleet's retry policy (3 attempts, fixed 1s delay, no jitter, no breaker, no budget) turned it into a 90-minute outage — synchronized retries tripled offered load, the provider's recovery attempts drowned repeatedly, and two customers were double-charged (timeout retries on a call whose idempotency keys had been "TODO" for a year). Rebuild: idempotency keys shipped FIRST (the incident's `regression-test-from-bug` — a test asserting one charge across a timeout-retry); error-class gate (429/503/timeout only, `Retry-After` honored — the provider had been SENDING it, ignored); full-jitter exponential (base 500ms, cap 8s, 2 attempts user-facing per the 10s UX budget arithmetic, written in the config comment); circuit breaker per provider (open at 50% failure over 30s, half-open probe at 15s — open-state fallback: the honest "payment processing delayed" state, `empty-loading-error-states` step 4); fleet retry budget at 10%. Chaos drill two weeks later (50% injected failures): breaker opened in 40s, load on the "provider" stayed under 1.2× baseline (vs the incident's 3×), zero duplicate charges. The real test came in month three — a genuine provider blip, 8 minutes: breaker opened, closed on the half-open probe, total customer impact 8 minutes of delayed-payment messaging, and the retry-rate dashboard had shown the degradation trend 6 minutes before the provider's own status page admitted it.

## Related skills
- `error-handling-strategy` — classifies what's retryable; this skill tunes the how.
- `tool-use-loop-debugging` — the transport-vs-semantic retry boundary in agents.
- `runaway-cost-guardrails` — the budget layer above agent-loop retries.
- `capacity-planning` — the retry-storm multiplier in load math.
- `data-pipeline-idempotency` — what makes retries safe to attempt at all.
