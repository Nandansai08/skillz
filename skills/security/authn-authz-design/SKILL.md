---
name: authn-authz-design
description: >
  Use when designing how a system authenticates and authorizes — sessions vs
  JWT, token lifetimes, RBAC vs ABAC, where authz checks live. Triggers:
  "sessions or JWT", "design the auth", "role-based access", "how should
  permissions work", "token expiry strategy", "auth for this API".
---

# AuthN/AuthZ Design

## When to use this skill
- Designing authentication or permissions for a new app/API, or overhauling one that grew ad hoc.
- Debating sessions vs tokens, or roles vs policies.
- NOT for reviewing an implementation for bugs — that's `secure-code-review` step 1; this is the design that makes those bugs structurally harder.

## Prerequisites
- The actor inventory: humans (browser? mobile?), services, third-party API consumers — each may need a different mechanism.
- Data sensitivity and any compliance constraints (they set session/token lifetime ceilings).

## Workflow

1. **Don't build authentication if you can rent it.** Managed identity (Auth0/Cognito/Entra/Keycloak self-hosted, or "Sign in with" federation) outperforms homegrown on the parts that kill you: credential storage, MFA, reset flows, breach detection. Build only the authorization layer, which is genuinely yours. If you must store passwords: argon2id/bcrypt, breach-list checks, MFA offered — and that's a project, not a feature.

2. **Web session mechanism — default answer: server-side sessions in httpOnly cookies.** Cookie flags: `HttpOnly; Secure; SameSite=Lax` minimum. Sessions are revocable instantly, small, and boring — correct properties for a first-party web app. JWTs earn their place where their actual advantage applies: *stateless verification across services* — the edge/gateway validates once, internal services verify the signature without a session-store round trip.

3. **If JWTs: constrain the sharp edges by design.** Access tokens short-lived (5–15 min) + refresh tokens (revocable, rotated on use, reuse-detection kills the family); algorithm pinned server-side (never honored from the token header — `alg:none` and RS256→HS256 confusion are the classic breaks); validate `iss`, `aud`, `exp` always; keep claims minimal (roles-in-token go stale for the token's lifetime — that's the revocation problem restated). Accept that "logout" means "wait out the access-token TTL" or maintain a denylist, which quietly reinvents sessions.

4. **Choose the authz model by rule shape, and start simpler than you think:**
   - **RBAC** (roles → permission sets): right when rules are "managers can approve" — most B2B apps. Keep roles ≤ ~7 before reaching for more machinery; role explosion ("regional-manager-readonly-eu") is the sign you've outgrown it.
   - **ABAC/policy** (attributes: owner, tenant, amount, time): right when rules sound like "users can edit *their own* drafts" or "approve invoices *under $10k*". Note: almost every app needs at least `owner == current_user`, so real systems are RBAC + resource-ownership checks — design that pair explicitly rather than pretending pure RBAC.
   - Centralize evaluation: one `authorize(actor, action, resource)` choke point (library or policy engine — OPA/Cedar/casbin when policies need to be data). Fifty inline `if user.role ==` checks is how the DELETE handler gets forgotten.
   - **Deny by default:** unlisted action = forbidden. Every endpoint opts *into* access; middleware enforces authentication globally with an explicit public-allowlist, not per-route remembering.

5. **Design tenancy isolation below the app layer where possible:** tenant_id in every query via scoped default querysets / row-level security (Postgres RLS) rather than developer discipline. Cross-tenant IDOR is the most common real-world authz failure; make it require *effort* instead of an oversight.

6. **Service-to-service and API consumers separately:** workload identity (mTLS/SPIFFE, cloud IAM) or client-credentials OAuth for services — never shared static API keys where avoidable (`secrets-management` step 1); scoped, per-consumer keys with rotation for third parties. Human auth mechanisms don't stretch to machines well.

7. **Design the auditable exhaust:** authn events (login, failure, MFA, refresh-reuse-detected) and authz denials logged with actor + action + resource — denial spikes are your IDOR-probing alarm. And rate-limit the auth surface itself (login, reset, token endpoints) — it's the most attacked square meter of the system.

## Common pitfalls
- JWT chosen "because modern" for a monolith with one web frontend — inheriting the revocation problem and localStorage-XSS temptation for zero stateless benefit. (Corollary: tokens in localStorage; httpOnly cookies exist for this.)
- Long-lived access tokens because refresh flows are annoying. A stolen 30-day token is a 30-day breach; the annoyance IS the security property.
- Roles checked in the frontend, API trusts the caller. The API is the boundary; UI checks are UX.
- Permissions accreted per-feature (`can_export_v2_reports`) until nobody can answer "what can support staff do?" — audit requires the model to be enumerable (step 4's choke point gives you this).
- Multi-tenancy by discipline: every query hand-remembering `tenant_id=`. One forgotten filter = data breach. Step 5 exists because discipline fails.
- Admin backdoors ("superuser bypasses authz") unlogged — the account that most needs auditing has the least.

## Example
B2B invoicing SaaS design. Chosen: Keycloak for authn (SSO required by customers anyway), server-side sessions for the web app, short-lived JWTs only on the public API gateway → internal hops. Authz: RBAC with 4 roles (admin, finance, member, readonly) + ownership/tenancy as ABAC conditions, evaluated through one `authorize()` middleware backed by Cedar policies; Postgres RLS on tenant_id as the backstop. Third-party API: per-consumer scoped keys, 90-day rotation. First pen test: zero cross-tenant findings (RLS caught the one query a dev forgot to scope — it returned empty instead of leaking), one finding on a missing rate limit for token refresh, fixed in a day.

## Related skills
- `secure-code-review` — auditing the implementation of this design.
- `threat-modeling` — where authn/authz requirements surface from.
- `secrets-management` — handling the keys/credentials this design mints.
- `api-design-rest` — the 401/403/404 surface these decisions produce.
