---
name: secrets-incident-response
description: >
  Use the moment a credential has leaked — committed to git, pasted in
  chat, exposed in logs or a public repo. Rotate-first playbook, usage
  audit, history purge. Triggers: "committed a secret", "API key leaked",
  "token in the repo", "password in Slack/logs", "found credentials in a
  public repo". NOT for designing secret storage (see secrets-management)
  — run this playbook first, fix the system after.
---

# Secrets Incident Response

## Overview

A leaked secret is compromised from the moment of exposure — rotation is the only remediation, and everything else (deleting the commit, purging history) is hygiene that follows. The usage audit is what separates "leak, contained" from "breach, unknown."

## When to Use

- A secret (API key, password, token, private key, connection string) has been exposed anywhere it shouldn't be — even briefly, even "private".
- A scanner or GitHub alert flags a committed credential.

**When NOT to use:**
- Designing storage/rotation systems — `secrets-management`, after this playbook completes.

## Prerequisites

- Ability to issue and revoke the credential (or the contact who can — find this NOW if unknown, it's the critical path).

## The Workflow

1. **Rotate first. Everything else is second.** Treat the secret as compromised from the moment of exposure — not "if someone saw it." Order for availability: **issue new → cut over → revoke old** (two-key overlap). Exception: active exploitation observed → revoke immediately and eat the outage.

2. **While rotating, freeze the spread — don't delete the evidence yet.** Make the exposed location non-public where possible (flip repo private, delete the chat message *after* screenshotting for the timeline), but keep local copies of relevant history for the audit. Note the exact exposure window: first-exposed timestamp → revocation timestamp. Inventory ALL exposure locations — the same key in the Slack thread, the Jira ticket, the log aggregator's index.

3. **Audit usage across the exposure window.** Provider audit logs (CloudTrail for the key's API calls, GitHub token audit, Stripe/Twilio dashboards): any calls from unfamiliar IPs/regions? Any calls at all, if the key was supposed to be dormant? Escalate to full incident (`incident-triage`) at the first sign of unauthorized use — a used leaked key is a breach, not a leak, and may carry disclosure obligations (loop in security/legal for customer-data access).

4. **Assess the blast radius of what that credential could reach.** What did it grant (scopes, roles)? What *else* trusts it (shared across services?)? Could it mint other credentials (an AWS key that can create keys, a CI token that can read other secrets)? Every reachable secondary credential joins the rotation list. Over-scoped keys turn one leak into many.

5. **Purge the history — after rotation, for hygiene not safety.**
   ```bash
   # git history rewrite (coordinate with the team first — rewrites all clones)
   git filter-repo --replace-text <(echo 'THE_LEAKED_VALUE==>REDACTED')
   git push --force-with-lease  # all forks/clones must re-clone
   ```
   Know the limits: GitHub caches commits reachable by SHA even after force-push (contact support to purge), forks keep copies, CI logs and package registries may hold it too. This is why rotation, not purging, is the safety step. Sequence matters: rotate → audit → then rewrite — a mid-audit force-push destroys the trail you're reading.

6. **Check for siblings before closing:** run `gitleaks`/`trufflehog` over the full history of the affected repo (and the author's other repos) — secrets travel in packs; the committed `.env` rarely held just one.

7. **Fix the class, blamelessly.** Mini-postmortem (`blameless-postmortem` discipline): how did it get there (no pre-commit hook? no push protection?), how long until detected (scanner gap?), how long to rotate (if hours: rotation drills are the action item). Ship at minimum: push protection + secret scanning enabled, pre-commit hook in the repo template, rotation runbook updated with what you just learned.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "I deleted the commit within minutes — nobody saw it" | Public-repo secrets get harvested by bots in under five minutes, and the value persists in reflogs, forks, and scrapers' databases regardless of your delete. Rotation or nothing. |
| "It was only exposed briefly, and the repo is obscure" | Exposure windows are measured by scanners, not by human attention. 'Obscure' is not an access control; the 33-minute window in the example produced four unauthorized calls. |
| "Rotation can wait until Monday — it's the weekend" | The attacker's calendar disagrees. Every hour of a live leaked credential is an hour of unaudited access; the two-key overlap makes rotation a low-risk operation any day. |
| "We rotated — done, no need to dig through logs" | Skipping the usage audit converts a possible breach into an unknown one. The interesting question was never 'is it fixed' but 'was it used' — and disclosure obligations may hang on the answer. |
| "Revoke it right now, worry about the service later" | Revoke-before-reissue self-inflicts an outage on top of the incident — unless exploitation is ACTIVE, two-key overlap does both safely. |
| "Find who committed it — this can't happen again" | The absence of push protection, hooks, and scanning is the system gap; the person is just who found it. Blame guarantees the next leak gets hidden instead of reported. |

## Red Flags

- The leak "handled" by deleting the commit/message, credential unrotated.
- No exposure-window timestamps recorded.
- Provider audit logs never pulled.
- History force-pushed mid-audit, trail destroyed.
- Sibling scan skipped ("it was just the one key").
- Same repo, second leak, still no push protection.

## Verification

- [ ] Credential rotated: new issued, consumers cut over, old revoked — timestamps recorded.
- [ ] Exposure window documented (first-exposed → revoked) with all exposure locations inventoried.
- [ ] Usage audit completed across the window — log query results attached; escalation decision recorded.
- [ ] Blast radius assessed; secondary credentials rotated where reachable — list included.
- [ ] History purged (with limits acknowledged) AFTER audit — rewrite noted, team coordinated.
- [ ] Sibling scan run over full history — output clean or findings handled.
- [ ] Class fixes shipped: push protection + scanning + hook — settings linked; mini-postmortem done.

## Example

14:20 — GitHub secret-scanning alert: AWS access key in a public repo, pushed 13:47. 14:25: new key issued, deploy started; old key's CloudTrail pulled in parallel. 14:31: old key revoked (33-min exposure window). Audit: 4 calls at 14:11 from an unfamiliar IP — `ListBuckets`, `GetCallerIdentity`, two `GetObject` on a config bucket. Escalated to incident: bucket contents reviewed (contained a DB connection string → secondary rotation of the DB password), security lead looped in, determined no customer data reachable. Sibling scan found a dormant Slack token from 2024 — rotated. Follow-ups: push protection org-wide, the key was IAM-admin-scoped for no reason → rescoped (`least-privilege-review`), rotation drill added quarterly. The 4 unauthorized calls, caught because someone actually read the audit log, changed this from "oops" to "contained breach."

## Related skills

- `secrets-management` — the system fixes after the fire is out.
- `incident-triage` — the escalation path when usage is found.
- `blameless-postmortem` — step 7's format.
- `least-privilege-review` — shrinking the next leak's blast radius.
