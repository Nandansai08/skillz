---
name: secrets-management
description: >
  Use when handling credentials in code, CI, or infrastructure — where
  secrets live, how they reach runtime, rotation, preventing git commits
  of secrets. Triggers: "where do I put this API key", "secrets in CI",
  "env vars vs vault", ".env file", "rotate credentials". NOT for
  responding to an already-leaked secret — that's secrets-incident-response,
  run it first, then return here to fix the system.
---

# Secrets Management

## Overview

The best secret is one that doesn't exist (workload identity), the second-best lives in a manager with audited access, and every other arrangement is a leak with a fuse. Rotation practiced as routine is what makes the eventual incident an errand instead of a crisis.

## When to Use

- Adding a credential to a service, pipeline, or dev setup.
- Auditing how an app currently gets its secrets.

**When NOT to use:**
- An active leak — `secrets-incident-response` first; system fixes after.

## Prerequisites

- Inventory of the secret in question: what it grants, who/what needs it, blast radius if leaked.

## The Workflow

1. **First: try to not have the secret.** Cloud-native identity beats stored credentials — IAM roles / workload identity for service→cloud auth, OIDC federation for CI→cloud (`id-token: write` in GitHub Actions instead of a stored `AWS_SECRET_ACCESS_KEY`), managed identities on Azure/GCP. A secret that doesn't exist can't leak. Only proceed when a literal secret is unavoidable (third-party API keys, DB passwords on legacy systems).

2. **Pick storage by tier:**
   - **Production:** a secrets manager (AWS Secrets Manager/SSM, GCP Secret Manager, Vault) — audited access, rotation hooks, IAM-scoped reads.
   - **CI:** the CI's secret store (GitHub environments-scoped secrets), never repo variables, never encrypted-in-repo blobs.
   - **Local dev:** `.env` file that is gitignored *and* only ever holds dev-grade credentials — a leaked dev secret should matter zero. Production values never touch laptops.
   Warning: Kubernetes Secrets are base64 in etcd unless encryption-at-rest is configured — base64 is not encryption. Terraform providers write credentials into state in plaintext — state goes in an encrypted, access-controlled backend.

3. **Deliver to runtime without persisting.** Injected env vars at start or mounted files (K8s secrets/CSI driver) are fine; what's forbidden: baked into images (see `dockerfile-optimization`), in config files under version control, in CLI args (visible in `ps`), in application logs (add a redaction filter to the logger — key names matching `(?i)(password|token|secret|key)`).

4. **Scope minimally and separately.** Per-environment secrets (staging key can't touch prod), per-service DB users, API keys with the narrowest scopes the vendor offers. One shared "the API key" across services means one leak rotates everything and audit logs identify nothing.

5. **Make rotation a routine operation, not a crisis skill.** Two-key overlap pattern: issue key B, deploy consumers reading B, revoke A. If rotating a secret requires a coordinated multi-team fire drill, that's the finding — fix the deploy path until rotation is boring, because the leak day rehearses whatever you practiced.

6. **Block commits mechanically.** Pre-commit hook + CI scan with `gitleaks` or `trufflehog`, and GitHub push protection enabled. The gitignored `.env` protects one path; scanners protect against the hardcoded-in-source path, which is where real leaks come from. Include `.env.example` files in the scan — they accumulate real values when someone copies the wrong direction.

7. **Audit reads.** Secrets-manager access logs feed the SIEM/alerting: a prod DB credential read by an unfamiliar principal is an incident signal. This is the payoff of step 2 — env vars can't tell you who read them.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "It's a private repo — a committed key is fine" | Private repos get forked, cloned to laptops, and opened to contractors; scanners harvest leaked private repos too. And the key outlives every one of those access decisions. |
| "We'll rotate it when we have time" | Unpracticed rotation means the leak-day rotation takes six hours of discovery under fire. The quarterly drill is an hour; it's the difference between errand and incident. |
| "One org-wide API key is simpler to manage" | Simpler until it leaks: now every service rotates at once, and the audit log attributes usage to nobody. Per-service scoping is the cost of a knowable blast radius. |
| "Base64 in the manifest is basically encrypted" | Base64 is encoding — `base64 -d` is the attack. Encryption-at-rest and access control are separate, real steps. |
| "OIDC setup is complex; a stored cloud key works today" | The stored key is a standing liability that OIDC deletes outright. One afternoon of federation config removes an entire leak class permanently — the best ratio in this domain. |
| "Push protection blocks developers — too much friction" | The friction is one override prompt on a false positive; the alternative is `secrets-incident-response` at 2pm on a Friday. Every team that's run that playbook enables push protection after. |

## Red Flags

- Any credential in git history, `docker history`, or CI logs — even "old" ones.
- `.env.example` containing values that work.
- A rotation that would require a meeting to schedule.
- The same token name appearing in multiple services' configs.
- Secrets-manager in place but read-audit logs feeding nothing.
- Prod credentials known to work from developer laptops.

## Verification

- [ ] Identity-over-secret checked first: OIDC/workload-identity used where the platform supports it — config linked; remaining literal secrets each justified.
- [ ] Storage per tier confirmed: no secrets in repo vars, images, or committed files — gitleaks scan output clean, linked.
- [ ] Log-redaction filter live — a test log line with a fake token shows redaction.
- [ ] Scopes per-service/per-env — the secret inventory lists each credential's blast radius.
- [ ] Rotation executed at least once via the documented path — runbook + last-rotated date recorded.
- [ ] Push protection + pre-commit + CI scanning all enabled — settings/screenshots linked.

## Example

Audit of a payments service found: AWS keys as GitHub repo secrets (long-lived, admin-scoped), prod DB password in `values.yaml` in git history, and Stripe key in a Docker ENV layer. Fixes: CI → OIDC federation with a deploy-scoped role (AWS keys deleted entirely per step 1); DB password moved to AWS Secrets Manager with per-env values and the git history handled via `secrets-incident-response`; Stripe key runtime-injected and restricted-scope reissued; gitleaks in pre-commit + CI; rotation runbook written and exercised once — rotation now takes 15 minutes.

## Related skills

- `secrets-incident-response` — when a secret has already leaked.
- `github-actions-authoring` — CI-side mechanics (OIDC, fork-PR exposure).
- `least-privilege-review` — the scoping discipline of step 4 applied broadly.
