---
name: least-privilege-review
description: >
  Use when auditing and shrinking permissions — IAM roles, service accounts,
  API scopes, database grants — down to what's actually used. Triggers:
  "audit our IAM", "this role has too many permissions", "least privilege",
  "what can this service account do", "tighten permissions", "access review".
---

# Least Privilege Review

## When to use this skill
- Periodic access reviews, post-incident scope-tightening, or before a compliance audit.
- A new service needs permissions and the temptation is `AdministratorAccess`.
- NOT for designing the app-level permission model — that's `authn-authz-design`; this is the infrastructure/platform layer beneath it.

## Prerequisites
- Access to the IAM/grants inventory AND usage logs (CloudTrail, GCP policy analyzer, DB query logs, API-key usage dashboards) — granted-vs-used is the whole method.

## Workflow

1. **Rank identities by blast radius, review in that order.** Not alphabetically, not by team: admin-equivalent roles first, then anything that can *mint credentials or modify IAM* (privilege-escalation paths — `iam:CreateAccessKey`, `iam:PassRole` with broad roles, service-account-token creation), then internet-facing services' identities, then CI/CD (it deploys everything, it's admin with extra steps), then humans, then the long tail.

2. **Diff granted against used.** The core move. Cloud-native tooling does the heavy lifting: AWS IAM Access Analyzer's "last accessed" data / `GenerateServiceLastAccessedDetails`, GCP IAM Recommender (it literally proposes the shrunken role), Azure PIM insights. For databases: grants vs `pg_stat_statements`-style query history. Anything unused for 90 days is a removal candidate — with the caveat of step 4.

3. **Kill the four classic over-grants on sight:**
   - Wildcards: `"Action": "s3:*"` where usage shows `GetObject` on one bucket; `"Resource": "*"` anywhere on a service that supports ARNs.
   - Managed convenience roles (`PowerUser`, project `Editor`, DB `OWNER`) standing in for ten specific permissions.
   - Human accounts with standing admin — replace with just-in-time elevation (short-lived, logged, approved: PIM/SSO role assumption) so admin is an *event*, not a state.
   - Shared identities (one service account for six services, team-shared logins) — unattributable and unshrinkable; split before you can scope (`secrets-management` step 4).

4. **Shrink with a safety net, not a cliff.** Sequence per identity: propose the reduced policy from usage data → run it *alongside* in audit/dry-run where the platform supports it (IAM policy simulation, `sudo` logging modes) or apply in staging first → apply in prod with the old policy one revert away → watch error rates for the identity's principal (AccessDenied spikes) for a week. Beware the 90-day trap: quarterly jobs, disaster-recovery paths, and year-end processes look unused right up until they page you — check schedules and runbooks before deleting their permissions.

5. **Fix the pipeline, not just the snapshot.** Findings recur unless the granting path changes: permissions defined in IaC so grants get code review (`infra-as-code-review` step 4 catches the wildcards at PR time), a paved-road module of pre-scoped roles for common service shapes (the reason people grab `*` is that scoping is annoying — make narrow the easy path), and new-grant requests requiring the resource list, not the service name.

6. **Handle the escalation graph explicitly.** A tightly-scoped identity that can `PassRole` a broad role, write to the Lambda another role executes, or edit the CI pipeline *has* the broad permissions transitively. Tools: `pmapper`/Cartography-style graph analysis for AWS, or at minimum grep policies for the known escalation primitives. This is where "we scoped everything" audits still fail pen tests.

7. **Schedule it and measure it.** Quarterly for the step-1 top tier, annually for the tail; track two numbers over time: identities with wildcard/admin grants (should trend to a named, justified handful) and median unused-permission count (should trend toward zero). One-time cleanups regrow within a year — the calendar is the control.

## Common pitfalls
- Reviewing humans and forgetting workloads. Service accounts outnumber humans 10:1 in most clouds and hold the scarier permissions; the CI role is the crown jewel most orgs never audit.
- Scoping actions but not resources: `s3:GetObject` on `"Resource": "*"` still reads every bucket including the backup vault. Both axes, always.
- Deleting "unused" access that belonged to the quarterly billing job — the step-4 trap. The incident this causes discredits the whole program, so the schedule check earns its minute.
- Treating deny-lists as scoping ("admin minus the dangerous stuff") — new permissions ship into the allow-side monthly; explicit allows only.
- Auditing direct grants while group memberships, inherited folder/org policies, and resource-based policies (bucket policies, KMS key policies) carry the real access. Review *effective* permissions, which is what the analyzer tools compute.
- Compliance-theater reviews: a spreadsheet of "manager approves continued access" rubber-stamps. Usage-diff evidence or it didn't happen.

## Example
Post-incident review (leaked key had admin — `secrets-incident-response` referral). Blast-radius ranking put 3 identities on top: CI deploy role (`AdministratorAccess`), the main API's service role (`s3:*`, `dynamodb:*` on `*`), and 11 humans with standing admin. Usage diff over 90 days: CI actually used 34 actions on 12 resource families → replaced with a generated policy + `iam:PassRole` locked to two named roles (killing the escalation path pmapper found); API role used `GetObject/PutObject` on two buckets and 6 DynamoDB actions on 3 tables → scoped exactly, deployed via a week of Access Analyzer dry-run, zero AccessDenied events; humans cut to 2 break-glass accounts (alarmed on use) + JIT elevation for the rest. Quarterly job trap avoided once: the "unused" `glacier:InitiateJob` belonged to the annual compliance export — found in a runbook, kept, documented. Next quarter's numbers: wildcard identities 14 → 2.

## Related skills
- `secrets-incident-response` — why blast radius keeps coming up.
- `infra-as-code-review` — the PR-time gate for new grants.
- `authn-authz-design` — the application-layer counterpart.
- `secrets-management` — identity-over-credentials, and per-service splitting.
