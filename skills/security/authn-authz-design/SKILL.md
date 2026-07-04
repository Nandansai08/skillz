---
name: authn-authz-design
description: >
  Use when designing how a system authenticates and authorizes — sessions
  vs JWT, token lifetimes, RBAC vs ABAC, where authz checks live.
  Triggers: "sessions or JWT", "design the auth", "role-based access",
  "how should permissions work", "token expiry strategy", "auth for this
  API". NOT for reviewing an implementation for bugs (see
  secure-code-review) — this is the design that makes those bugs
  structurally harder.
---

# AuthN/AuthZ Design

## Overview

Auth design is where security is cheapest: rent authentication instead of building it, default to boring sessions over fashionable tokens, centralize authorization in one choke point, and push tenancy isolation below the app layer. Every one of these choices converts a class of future bugs into a structural impossibility.

## When to Use

- Designing authentication or permissions for a new app/API, or overhauling one that grew ad hoc.
- Debating sessions vs tokens, or roles vs policies.

**When NOT to use:**
- Hunting implementation bugs in existing auth code — `secure-code-review` step 1.

## Prerequisites

- The actor inventory: humans (browser? mobile?), services, third-party API consumers — each may need a different mechanism.
- Data sensitivity and any compliance constraints (they set session/token lifetime ceilings).

## The Workflow

1. **Don't build authentication if you can rent it.** Managed identity (Auth0/Cognito/Entra/Keycloak self-hosted, or federation) outperforms homegrown on the parts that kill you: credential storage, MFA, reset flows, breach detection. Build only the authorization layer, which is genuinely yours. If you must store passwords: argon2id/bcrypt, breach-list checks, MFA offered — and that's a project, not a feature.

2. **Web session mechanism — default answer: server-side sessions in httpOnly cookies.** Cookie flags: `HttpOnly; Secure; SameSite=Lax` minimum. Sessions are revocable instantly, small, and boring — correct properties for a first-party web app. JWTs earn their place where their actual advantage applies: *stateless verification across services* — the edge validates once, internal services verify the signature without a session-store round trip.

3. **If JWTs: constrain the sharp edges by design.** Access tokens short-lived (5–15 min) + refresh tokens (revocable, rotated on use, reuse-detection kills the family); algorithm pinned server-side (never honored from the token header — `alg:none` and RS256→HS256 confusion are the classic breaks); validate `iss`, `aud`, `exp` always; keep claims minimal (roles-in-token go stale for the token's lifetime). Accept that "logout" means "wait out the access-token TTL" or maintain a denylist — which quietly reinvents sessions.

4. **Choose the authz model by rule shape, and start simpler than you think:**
   - **RBAC** (roles → permission sets): right when rules are "managers can approve" — most B2B apps. Keep roles ≤ ~7; role explosion ("regional-manager-readonly-eu") is the outgrown-it signal.
   - **ABAC/policy** (attributes: owner, tenant, amount): right when rules sound like "users edit *their own* drafts." Note: almost every app needs at least `owner == current_user`, so real systems are RBAC + ownership checks — design the pair explicitly.
   - Centralize evaluation: one `authorize(actor, action, resource)` choke point (library or policy engine — OPA/Cedar/casbin when policies need to be data). Fifty inline `if user.role ==` checks is how the DELETE handler gets forgotten.
   - **Deny by default:** unlisted action = forbidden. Middleware enforces authentication globally with an explicit public-allowlist, not per-route remembering.

5. **Design tenancy isolation below the app layer where possible:** tenant_id in every query via scoped default querysets / row-level security (Postgres RLS) rather than developer discipline. Cross-tenant IDOR is the most common real-world authz failure; make it require *effort* instead of an oversight.

6. **Service-to-service and API consumers separately:** workload identity (mTLS/SPIFFE, cloud IAM) or client-credentials OAuth for services — never shared static API keys where avoidable (`secrets-management` step 1); scoped, per-consumer keys with rotation for third parties. Human auth mechanisms don't stretch to machines well.

7. **Design the auditable exhaust:** authn events (login, failure, MFA, refresh-reuse-detected) and authz denials logged with actor + action + resource — denial spikes are your IDOR-probing alarm. Rate-limit the auth surface itself (login, reset, token endpoints) — the most attacked square meter of the system. And admin/superuser paths get MORE logging, not less: the account that most needs auditing usually has the least.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "JWTs are the modern standard — sessions are legacy" | For a monolith with one web frontend, JWTs import the revocation problem and the localStorage temptation for zero stateless benefit. Boring sessions are the senior choice there. |
| "Long-lived access tokens — refresh flows annoy users" | A stolen 30-day token is a 30-day breach with no revocation lever. The refresh dance IS the security property, not friction around it. |
| "The frontend hides buttons users can't use" | The API is the boundary; UI checks are UX. Every authz decision enforced only in the client is enforced nowhere. |
| "We'll add per-feature permission flags as we go" | Accretion produces `can_export_v2_reports` × 200, and 'what can support staff do?' becomes unanswerable — un-auditable by construction. The choke point keeps the model enumerable. |
| "Devs will remember to scope queries by tenant" | One forgotten WHERE clause = data breach. RLS/scoped querysets make the leak require effort instead of an oversight — discipline is not an architecture. |
| "Building login ourselves is a weekend project" | Credential storage, MFA, resets, breach detection, lockouts — the weekend version is missing exactly the parts that end up in disclosure letters. |

## Red Flags

- Tokens in localStorage.
- Auth logic sprinkled as inline role checks across handlers.
- A "temporary" admin backdoor with no logging.
- JWT libraries configured to accept the token's own `alg` header.
- Tenant scoping done per-query by convention, no RLS/default-scope backstop.
- One static API key shared by multiple services.
- No rate limiting on login/reset/token endpoints.

## Verification

- [ ] Authn rented or the build-it decision explicitly justified in an ADR — link.
- [ ] Session/token choice matched to the actor inventory with the trade-off stated — design doc linked.
- [ ] Single authorization choke point demonstrated: grep shows no scattered inline role checks in new code.
- [ ] Deny-by-default proven: an unregistered route/action returns 401/403 in a test — linked.
- [ ] Tenancy backstop below the app layer (RLS/scoped querysets) — config/migration linked; cross-tenant test returns empty/404.
- [ ] Authz denials and authn events visible in logs with actor+action+resource — sample log lines attached.
- [ ] Rate limits live on the auth surface — config linked.

## Example

B2B invoicing SaaS design. Chosen: Keycloak for authn (SSO required by customers anyway), server-side sessions for the web app, short-lived JWTs only on the public API gateway → internal hops. Authz: RBAC with 4 roles (admin, finance, member, readonly) + ownership/tenancy as ABAC conditions, evaluated through one `authorize()` middleware backed by Cedar policies; Postgres RLS on tenant_id as the backstop. Third-party API: per-consumer scoped keys, 90-day rotation. First pen test: zero cross-tenant findings (RLS caught the one query a dev forgot to scope — it returned empty instead of leaking), one finding on a missing rate limit for token refresh, fixed in a day.

## Related skills

- `secure-code-review` — auditing the implementation of this design.
- `threat-modeling` — where authn/authz requirements surface from.
- `secrets-management` — handling the keys/credentials this design mints.
- `api-design-rest` — the 401/403/404 surface these decisions produce.
