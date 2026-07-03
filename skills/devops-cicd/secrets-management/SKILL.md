---
name: secrets-management
description: >
  Use when handling credentials in code, CI, or infrastructure — where secrets
  live, how they reach runtime, rotation, and preventing git commits of
  secrets. Triggers: "where do I put this API key", "secrets in CI",
  "env vars vs vault", ".env file", "rotate credentials",
  "committed a secret" (for the leak itself see secrets-incident-response).
---

# Secrets Management

## When to use this skill
- Adding a credential to a service, pipeline, or dev setup.
- Auditing how an app currently gets its secrets.
- NOT for responding to an already-leaked secret — that's `secrets-incident-response`, go there first, then come back to fix the system.

## Prerequisites
- Inventory of the secret in question: what it grants, who/what needs it, blast radius if leaked.

## Workflow

1. **First: try to not have the secret.** Cloud-native identity beats stored credentials — IAM roles / workload identity for service→cloud auth, OIDC federation for CI→cloud (`id-token: write` in GitHub Actions instead of a stored `AWS_SECRET_ACCESS_KEY`), managed identities on Azure/GCP. A secret that doesn't exist can't leak. Only proceed when a literal secret is unavoidable (third-party API keys, DB passwords on legacy systems).

2. **Pick storage by tier:**
   - **Production:** a secrets manager (AWS Secrets Manager/SSM, GCP Secret Manager, Vault) — audited access, rotation hooks, IAM-scoped reads.
   - **CI:** the CI's secret store (GitHub environments-scoped secrets), never repo variables, never encrypted-in-repo blobs.
   - **Local dev:** `.env` file that is gitignored *and* only ever holds dev-grade credentials — a leaked dev secret should matter zero. Production values never touch laptops.

3. **Deliver to runtime without persisting.** Injected env vars at start or mounted files (K8s secrets/CSI driver) are fine; what's forbidden: baked into images (see `dockerfile-optimization`), in config files under version control, in CLI args (visible in `ps`), in application logs (add a redaction filter to the logger — key names matching `(?i)(password|token|secret|key)`).

4. **Scope minimally and separately.** Per-environment secrets (staging key can't touch prod), per-service DB users, API keys with the narrowest scopes the vendor offers. One shared "the API key" across services means one leak rotates everything and audit logs identify nothing.

5. **Make rotation a routine operation, not a crisis skill.** Two-key overlap pattern: issue key B, deploy consumers reading B, revoke A. If rotating a secret requires a coordinated multi-team fire drill, that's the finding — fix the deploy path until rotation is boring, because the leak day rehearses whatever you practiced.

6. **Block commits mechanically.** Pre-commit hook + CI scan with `gitleaks` or `trufflehog`, and GitHub push protection enabled. The gitignored `.env` protects one path; scanners protect against the hardcoded-in-source path, which is where real leaks come from.

7. **Audit reads.** Secrets-manager access logs feed the SIEM/alerting: a prod DB credential read by an unfamiliar principal is an incident signal. This is the payoff of step 2 — env vars can't tell you who read them.

## Common pitfalls
- `.env.example` accumulating real values over time because someone copied the wrong direction. Example files hold placeholder text only; add them to the gitleaks scan too.
- Secrets in Terraform state — many providers write credentials into state files in plaintext. State goes in an encrypted, access-controlled backend, and prefer data-source lookups over hardcoded values.
- Kubernetes Secrets assumed encrypted — they're base64 in etcd unless encryption-at-rest is configured. Base64 is not encryption.
- Rotation deferred until "we have time," so day-of-leak rotation takes 6 hours of discovery. Step 5's drill is the mitigation.
- CI secrets exposed to fork PRs via `pull_request_target` or overly broad environment rules — see `github-actions-authoring` step 1.
- One org-wide `SLACK_TOKEN` used by 14 services: unattributable usage, unrotatable without a coordination day.

## Example
Audit of a payments service found: AWS keys as GitHub repo secrets (long-lived, admin-scoped), prod DB password in `values.yaml` in git history, and Stripe key in a Docker ENV layer. Fixes: CI → OIDC federation with a deploy-scoped role (AWS keys deleted entirely per step 1); DB password moved to AWS Secrets Manager with per-env values and the git history handled via `secrets-incident-response`; Stripe key runtime-injected and restricted-scope reissued; gitleaks in pre-commit + CI; rotation runbook written and exercised once — rotation now takes 15 minutes.

## Related skills
- `secrets-incident-response` — when a secret has already leaked.
- `github-actions-authoring` — CI-side mechanics (OIDC, fork-PR exposure).
- `least-privilege-review` — the scoping discipline of step 4 applied broadly.
