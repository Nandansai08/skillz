---
name: security-headers-config
description: >
  Use when configuring web security headers — CSP, CORS, HSTS, frame and
  content-type protections — what each stops and the safe rollout order.
  Triggers: "set up CSP", "CORS error" (the fixing-it-securely kind),
  "security headers", "HSTS", "clickjacking protection",
  "securityheaders.com says F".
---

# Security Headers Config

## When to use this skill
- Hardening a web app's response headers, or fixing a failing header audit.
- Unblocking a CORS problem *without* cargo-culting `*` origins.
- NOT for API authn/authz design — headers complement, never replace, server-side checks (`authn-authz-design`).

## Prerequisites
- Where headers get set for this stack (reverse proxy/CDN config, framework middleware like helmet/django-security) — set them at one layer, not three fighting layers.
- An inventory of legitimate cross-origin needs: script/style/img sources, API consumers, embedding requirements.

## Workflow

1. **Ship the uncontroversial floor first — near-zero breakage risk:**
   ```
   X-Content-Type-Options: nosniff
   X-Frame-Options: DENY            # or CSP frame-ancestors (step 3); keep both for old browsers
   Referrer-Policy: strict-origin-when-cross-origin
   Permissions-Policy: camera=(), microphone=(), geolocation=()   # deny what you don't use
   ```
   These stop MIME-sniffing attacks, clickjacking, referrer leakage of URLs/tokens, and drive-by feature access respectively. Deploy today.

2. **HSTS — powerful, and the one header that can lock you out:**
   ```
   Strict-Transport-Security: max-age=31536000; includeSubDomains
   ```
   It pins browsers to HTTPS for `max-age` seconds — which means an HTTP-only subdomain (that old internal tool on `legacy.example.com`) breaks *for the full duration* once `includeSubDomains` ships. Rollout: audit subdomains → `max-age=300` for a week → raise to a year → consider `preload` only when you accept it's effectively permanent (removal from the preload list takes months).

3. **CSP — the big one; roll out in report-only, always.** Purpose: even if HTML injection lands, the script doesn't run. Realistic modern policy (nonce-based, not allowlist-based — allowlists of CDNs are widely bypassable):
   ```
   Content-Security-Policy-Report-Only:
     default-src 'self';
     script-src 'nonce-{random}' 'strict-dynamic';
     object-src 'none'; base-uri 'none'; frame-ancestors 'none';
     report-uri /csp-reports
   ```
   Process: report-only + a report endpoint → run 1–2 weeks of real traffic → triage reports (each is either a policy gap to allow or an inline-script to refactor onto nonces) → enforce. Third-party scripts (analytics, chat widgets) are where the pain lives; `'strict-dynamic'` lets nonced scripts load their children, which handles most of it.

4. **CORS — configure as an allowlist of origins, never reflect blindly.** CORS *relaxes* the browser's same-origin protection; every relaxation is a grant.
   - `Access-Control-Allow-Origin`: exact origins from a server-side allowlist. Never `*` with credentials (browsers block it — which is the spec saving you); never echo the request's Origin header back unvalidated (equivalent to `*` with credentials — a real, common vuln).
   - `Allow-Credentials: true` only if cookies/auth actually cross origins.
   - The debugging rule: a CORS error means the *browser* is protecting the user; the fix is adding the specific legitimate origin server-side — not disabling the protection, not a `cors: true` middleware default.

5. **Cookies get their armor in the same pass** (they're headers too): `Secure; HttpOnly; SameSite=Lax` baseline (`authn-authz-design` step 2), `SameSite=Strict` for the sensitive actions, `__Host-` prefix to pin path/domain.

6. **Verify from outside:** `curl -sI https://yoursite | grep -iE 'strict|content-sec|x-frame|x-content'`, then securityheaders.com / Mozilla Observatory for the graded pass. Check *every* response class — error pages, redirects, and static-asset responses often bypass the middleware that decorates HTML responses.

7. **Monitor the exhaust:** CSP reports keep flowing post-enforcement — spikes mean either a deploy broke the nonce plumbing or someone's probing. Route them somewhere with a dashboard, sample if noisy (browser extensions generate junk reports; filter known-extension patterns).

## Common pitfalls
- CSP with `'unsafe-inline'` in script-src "temporarily" — that's ~90% of CSP's value gone; the temporary lasts years. Nonces + the report-only migration path exist precisely for this.
- Deploying CSP enforce-mode straight to prod on a Friday: the checkout button's inline handler stops working, revenue graph goes flat. Report-only first, always.
- HSTS preload submitted during the hardening sprint, discovered when the acquisition's HTTP-only subdomain can't load — treat preload as irreversible.
- Origin-reflection CORS middleware (`res.set('A-C-A-O', req.headers.origin)`) shipped to "fix CORS errors" — silently grants every website credentialed access to your API.
- Headers set in the framework but the CDN serves error pages/static assets without them — attackers use whichever response lacks the armor. Verify per response class (step 6).
- Treating headers as the defense rather than depth: CSP mitigates the XSS you failed to prevent; output encoding (`input-validation-boundaries` step 2) is still the primary control.

## Example
B2B app, securityheaders.com grade F, one prior stored-XSS incident. Week 1: floor headers + cookie flags shipped (zero breakage), HSTS at `max-age=300`. Weeks 2–3: CSP report-only revealed 23 inline scripts, one `javascript:` href, and a marketing tag manager injecting arbitrary children — inline scripts moved to nonced files, tag manager wrapped with `'strict-dynamic'`. Week 4: CSP enforced; HSTS raised to one year after the subdomain audit killed one HTTP-only legacy host. CORS audit found the origin-reflection pattern in the API gateway — replaced with a 3-origin allowlist; one partner integration surfaced as legitimately broken and was added explicitly. Grade A; the next XSS attempt (six months later, via a markdown field) executed nothing — CSP report showed the blocked attempt, which is how they learned about the markdown bug.

## Related skills
- `input-validation-boundaries` — the primary XSS control CSP backs up.
- `authn-authz-design` — the cookie/session decisions step 5 hardens.
- `secure-code-review` — hunting the XSS sinks themselves.
