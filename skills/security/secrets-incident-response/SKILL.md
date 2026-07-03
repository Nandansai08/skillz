---
name: secrets-incident-response
description: >
  Use the moment a credential has leaked — committed to git, pasted in chat,
  exposed in logs or a public repo. Rotate-first playbook, usage audit,
  history purge. Triggers: "committed a secret", "API key leaked", "token
  in the repo", "password in Slack/logs", "found credentials in a public repo".
---

# Secrets Incident Response

## When to use this skill
- A secret (API key, password, token, private key, connection string) has been exposed anywhere it shouldn't be — even briefly, even "private".
- A scanner or GitHub alert flags a committed credential.
- NOT for designing secret storage — that's `secrets-management`; run this playbook first, fix the system after.

## Prerequisites
- Ability to issue and revoke the credential (or the contact who can — find this NOW if unknown, it's the critical path).

## Workflow

1. **Rotate first. Everything else is second.** Treat the secret as compromised from the moment of exposure — not "if someone saw it." Issue the new credential, deploy consumers to use it, revoke the old one. Order matters for availability: **issue new → cut over → revoke old** (two-key overlap, `secrets-management` step 5). Exception: active exploitation observed → revoke immediately and eat the outage.

2. **While rotating, freeze the spread — don't delete the evidence yet.** Make the exposed location non-public where possible (flip repo private, delete the chat message *after* screenshotting for the timeline), but keep local copies of relevant history for the audit. Note the exact exposure window: first-exposed timestamp → revocation timestamp.

3. **Audit usage across the exposure window.** Provider audit logs (AWS CloudTrail for the key's API calls, GitHub token audit, Stripe/Twilio dashboards): any calls from unfamiliar IPs/regions? Any calls at all, if the key was supposed to be dormant? Escalate to full incident (`incident-triage`) at the first sign of unauthorized use — a used leaked key is a breach, not a leak, and may carry disclosure obligations (loop in security/legal for customer data access).

4. **Assess the blast radius of what that credential could reach.** What did it grant (scopes, roles)? What *else* trusts it (shared across services? — this is why `secrets-management` step 4 wants per-service keys)? Could it mint other credentials (an AWS key that can create keys, a CI token that can read other secrets)? Every reachable secondary credential joins the rotation list. Over-scoped keys turn one leak into many.

5. **Purge the history — after rotation, for hygiene not safety.** A rotated key in history is dead; purge anyway (scrapers harvest patterns, and dead keys reveal naming conventions):
   ```bash
   # git history rewrite (coordinate with the team first — rewrites all clones)
   git filter-repo --replace-text <(echo 'THE_LEAKED_VALUE==>REDACTED')
   git push --force-with-lease  # all forks/clones must re-clone
   ```
   Know the limits: GitHub caches commits reachable by SHA even after force-push (contact GitHub support to purge), forks keep copies, CI logs and package registries may hold it too. This is why rotation, not purging, is the safety step.

6. **Check for siblings before closing:** run `gitleaks`/`trufflehog` over the full history of the affected repo (and the author's other repos) — secrets travel in packs; the committed `.env` rarely held just one.

7. **Fix the class, blamelessly.** Mini-postmortem (`blameless-postmortem` discipline): how did it get there (no pre-commit hook? no push protection? real value in `.env.example`?), how long until detected (scanner gap?), how long to rotate (if hours: rotation drills are the action item). Ship at minimum: push protection + secret scanning enabled, pre-commit hook in the repo template, and the rotation runbook updated with what you just learned.

## Common pitfalls
- Deleting the commit/message and calling it handled. The secret is in reflogs, forks, clones, CI caches, and scrapers' databases — GitHub public-repo secrets get tried by bots in *minutes*. Rotation is the only remediation; deletion is cosmetics.
- Rotating but never auditing usage. The interesting question isn't "did we fix it" but "was it used" — skipping step 3 converts a possible breach into an unknown one.
- Revoke-before-reissue on a production credential: self-inflicted outage stacked on the incident. Two-key overlap unless actively exploited.
- Panic history-rewrite of a shared repo mid-rotation, force-pushing over colleagues' work and destroying the audit trail you needed for step 3. Sequence: rotate → audit → then rewrite, coordinated.
- Blaming the committer. The absence of push protection, scanning, and hooks is a system gap; the person is just the one who found it (`blameless-postmortem` step 4 reasoning).
- Forgetting non-git exposures: the same key in the Slack thread, the Jira ticket, the log aggregator's index. The exposure inventory is part of step 2.

## Example
14:20 — GitHub secret-scanning alert: AWS access key in a public repo, pushed 13:47. 14:25: new key issued, deploy started; old key's CloudTrail pulled in parallel. 14:31: old key revoked (33-min exposure window). Audit: 4 calls at 14:11 from an unfamiliar IP — `ListBuckets`, `GetCallerIdentity`, two `GetObject` on a config bucket. Escalated to incident: bucket contents reviewed (contained a DB connection string → secondary rotation of the DB password), security lead looped in, determined no customer data reachable. Sibling scan found a dormant Slack token from 2024 — rotated. Follow-ups: push protection org-wide, the key was IAM-admin-scoped for no reason → rescoped (`least-privilege-review`), rotation drill added quarterly. The 4 unauthorized calls, caught because someone actually read the audit log, changed this from "oops" to "contained breach."

## Related skills
- `secrets-management` — the system fixes after the fire is out.
- `incident-triage` — the escalation path when usage is found.
- `blameless-postmortem` — step 7's format.
- `least-privilege-review` — shrinking the next leak's blast radius.
