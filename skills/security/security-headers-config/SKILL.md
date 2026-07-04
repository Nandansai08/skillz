---
name: security-headers-config
description: >
  Use when configuring web security headers — CSP, CORS, HSTS, frame and
  content-type protections — what each stops and the safe rollout order.
  Triggers: "set up CSP", "CORS error" (fixing it securely), "security
  headers", "HSTS", "clickjacking protection", "securityheaders.com says
  F". NOT for API authn/authz design — headers complement, never
  replace, server-side checks (see authn-authz-design).
---

# Security Headers Config

## Overview

Headers are defense-in-depth with a rollout order: the uncontroversial floor ships today, HSTS ramps carefully (it can lock you out), CSP goes report-only-first always, and CORS is an allowlist, never a reflex. The order exists because two of these can break production in ways the other two can't.

## When to Use

- Hardening a web app's response headers, or fixing a failing header audit.
- Unblocking a CORS problem *without* cargo-culting `*` origins.

**When NOT to use:**
- Authn/authz — headers back up server-side checks; they never replace them.

## Prerequisites

- Where headers get set for this stack (reverse proxy/CDN config, framework middleware) — set them at one layer, not three fighting layers.
- An inventory of legitimate cross-origin needs: script/style/img sources, API consumers, embedding requirements.

## The Workflow

1. **Ship the uncontroversial floor first — near-zero breakage risk:**
   ```
   X-Content-Type-Options: nosniff
   X-Frame-Options: DENY            # or CSP frame-ancestors; keep both for old browsers
   Referrer-Policy: strict-origin-when-cross-origin
   Permissions-Policy: camera=(), microphone=(), geolocation=()
   ```
   These stop MIME-sniffing, clickjacking, referrer leakage, and drive-by feature access. Deploy today.

2. **HSTS — powerful, and the one header that can lock you out:**
   ```
   Strict-Transport-Security: max-age=31536000; includeSubDomains
   ```
   It pins browsers to HTTPS for `max-age` seconds — an HTTP-only subdomain breaks *for the full duration* once `includeSubDomains` ships. Rollout: audit subdomains → `max-age=300` for a week → raise to a year → `preload` only when you accept it's effectively permanent.

3. **CSP — the big one; roll out in report-only, always.** Purpose: even if HTML injection lands, the script doesn't run. Modern policy is nonce-based, not allowlist-based (CDN allowlists are widely bypassable):
   ```
   Content-Security-Policy-Report-Only:
     default-src 'self';
     script-src 'nonce-{random}' 'strict-dynamic';
     object-src 'none'; base-uri 'none'; frame-ancestors 'none';
     report-uri /csp-reports
   ```
   Process: report-only + report endpoint → 1–2 weeks of real traffic → triage reports (each is a policy gap to allow or an inline-script to refactor onto nonces) → enforce. `'strict-dynamic'` lets nonced scripts load their children, which handles most third-party pain.

4. **CORS — configure as an allowlist of origins, never reflect blindly.** CORS *relaxes* the browser's protection; every relaxation is a grant.
   - `Access-Control-Allow-Origin`: exact origins from a server-side allowlist. Never `*` with credentials; never echo the request's Origin back unvalidated (equivalent to `*` with credentials — a real, common vuln).
   - `Allow-Credentials: true` only if cookies/auth actually cross origins.
   - The debugging rule: a CORS error means the *browser* is protecting the user; the fix is adding the specific legitimate origin server-side — not disabling the protection.

5. **Cookies get their armor in the same pass:** `Secure; HttpOnly; SameSite=Lax` baseline (`authn-authz-design` step 2), `SameSite=Strict` for sensitive actions, `__Host-` prefix to pin path/domain.

6. **Verify from outside, on every response class:** `curl -sI` + securityheaders.com / Mozilla Observatory for the graded pass — and check error pages, redirects, and static-asset responses, which often bypass the middleware that decorates HTML responses. Attackers use whichever response lacks the armor.

7. **Monitor the exhaust:** CSP reports keep flowing post-enforcement — spikes mean either a deploy broke the nonce plumbing or someone's probing. Route them to a dashboard, filter known browser-extension junk.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "Add 'unsafe-inline' for now, we'll remove it later" | That's ~90% of CSP's value gone, and 'later' is measured in years. The report-only migration path exists precisely so you never need it. |
| "Ship CSP enforcing — we tested it on staging" | Staging doesn't run your marketing tag manager's third-party children or your users' browser extensions. Enforce-day breakage lands on the checkout button; report-only first, always. |
| "Reflect the Origin header back — fixes CORS for everyone" | 'Everyone' includes every malicious site: origin reflection with credentials is equivalent to `*` with credentials — you've granted the whole web authenticated API access. |
| "HSTS preload now — we're committed to HTTPS" | Preload is effectively permanent, and the acquisition's HTTP-only subdomain arrives next year. Ramp max-age; treat preload as a one-way door. |
| "Headers are set in the framework — done" | The CDN serves error pages and static assets without them. Per-response-class verification (step 6) is where header audits actually pass or fail. |
| "We have CSP, so XSS is handled" | CSP mitigates the XSS you failed to prevent; output encoding remains the primary control. Depth, not substitution. |

## Red Flags

- `'unsafe-inline'` in script-src of an enforced policy.
- CORS middleware echoing request origins with credentials enabled.
- HSTS with includeSubDomains shipped without a subdomain audit.
- Error pages returning none of the floor headers.
- CSP deployed enforcing on day one, rolled back by day two.
- No CSP report endpoint — enforcement flying blind.

## Verification

- [ ] Floor headers present on ALL response classes (HTML, errors, redirects, static) — curl outputs attached per class.
- [ ] HSTS rollout stage documented; subdomain audit results linked before includeSubDomains.
- [ ] CSP ran report-only ≥1 week; report triage summary linked before enforcement.
- [ ] CORS allowlist server-side and exact; reflection absent — config linked, tested with a foreign Origin.
- [ ] Cookie flags verified on the session cookie — response header shown.
- [ ] External grade recorded (securityheaders.com/Observatory) before/after.
- [ ] CSP reports flowing to a monitored destination post-enforcement.

## Example

B2B app, securityheaders.com grade F, one prior stored-XSS incident. Week 1: floor headers + cookie flags shipped (zero breakage), HSTS at `max-age=300`. Weeks 2–3: CSP report-only revealed 23 inline scripts, one `javascript:` href, and a marketing tag manager injecting arbitrary children — inline scripts moved to nonced files, tag manager wrapped with `'strict-dynamic'`. Week 4: CSP enforced; HSTS raised to one year after the subdomain audit killed one HTTP-only legacy host. CORS audit found the origin-reflection pattern in the API gateway — replaced with a 3-origin allowlist; one partner integration surfaced as legitimately broken and was added explicitly. Grade A; the next XSS attempt (six months later, via a markdown field) executed nothing — CSP report showed the blocked attempt, which is how they learned about the markdown bug.

## Related skills

- `input-validation-boundaries` — the primary XSS control CSP backs up.
- `authn-authz-design` — the cookie/session decisions step 5 hardens.
- `secure-code-review` — hunting the XSS sinks themselves.
