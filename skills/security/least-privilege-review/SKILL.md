---
name: least-privilege-review
description: >
  Use when auditing and shrinking permissions — IAM roles, service
  accounts, API scopes, database grants — down to what's actually used.
  Triggers: "audit our IAM", "this role has too many permissions", "least
  privilege", "what can this service account do", "tighten permissions",
  "access review". NOT for designing the app-level permission model (see
  authn-authz-design) — this is the infrastructure layer beneath it.
---

# Least Privilege Review

## Overview

Granted-versus-used is the whole method: cloud analyzers already know which permissions each identity exercised; the review's job is shrinking the gap with a safety net, killing the four classic over-grants, and fixing the granting pipeline so the findings don't regrow.

## When to Use

- Periodic access reviews, post-incident scope-tightening, or compliance prep.
- A new service needs permissions and the temptation is `AdministratorAccess`.

**When NOT to use:**
- Application-layer roles/policies — `authn-authz-design`.

## Prerequisites

- Access to the IAM/grants inventory AND usage logs (CloudTrail, GCP policy analyzer, DB query logs) — granted-vs-used is the whole method.

## The Workflow

1. **Rank identities by blast radius, review in that order.** Admin-equivalent roles first, then anything that can *mint credentials or modify IAM* (privilege-escalation paths — `iam:CreateAccessKey`, `iam:PassRole` with broad roles), then internet-facing services' identities, then CI/CD (it deploys everything — admin with extra steps), then humans, then the long tail.

2. **Diff granted against used.** The core move. Cloud-native tooling does the heavy lifting: AWS IAM Access Analyzer's last-accessed data, GCP IAM Recommender (it literally proposes the shrunken role), Azure PIM insights. For databases: grants vs query history. Anything unused for 90 days is a removal candidate — with the step-4 caveat.

3. **Kill the four classic over-grants on sight:**
   - Wildcards: `"Action": "s3:*"` where usage shows `GetObject` on one bucket; `"Resource": "*"` anywhere ARNs are supported — scope BOTH axes; actions-scoped-resources-wild still reads every bucket including the backup vault.
   - Managed convenience roles (`PowerUser`, project `Editor`, DB `OWNER`) standing in for ten specific permissions.
   - Human accounts with standing admin — replace with just-in-time elevation (short-lived, logged, approved) so admin is an *event*, not a state.
   - Shared identities (one service account for six services) — unattributable and unshrinkable; split before you can scope.

4. **Shrink with a safety net, not a cliff.** Sequence per identity: propose the reduced policy from usage data → dry-run/audit-mode where the platform supports it (IAM policy simulation) or staging first → apply in prod with the old policy one revert away → watch AccessDenied rates for a week. Beware the 90-day trap: quarterly jobs, disaster-recovery paths, and year-end processes look unused right up until they page you — check schedules and runbooks before deleting their permissions.

5. **Fix the pipeline, not just the snapshot.** Findings recur unless the granting path changes: permissions defined in IaC so grants get code review (`infra-as-code-review` step 4 catches wildcards at PR time), a paved-road module of pre-scoped roles for common service shapes (people grab `*` because scoping is annoying — make narrow the easy path), and new-grant requests requiring the resource list, not the service name.

6. **Handle the escalation graph explicitly.** A tightly-scoped identity that can `PassRole` a broad role, write to the Lambda another role executes, or edit the CI pipeline *has* the broad permissions transitively. Tools: pmapper/Cartography-style graph analysis, or at minimum grep policies for the known escalation primitives. This is where "we scoped everything" audits still fail pen tests. Review *effective* permissions — group memberships, inherited org policies, and resource-based policies (bucket/KMS policies) carry access direct grants don't show.

7. **Schedule it and measure it.** Quarterly for the step-1 top tier, annually for the tail; track two numbers over time: identities with wildcard/admin grants (trending to a named, justified handful) and median unused-permission count (trending toward zero). One-time cleanups regrow within a year — the calendar is the control.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "The service needs admin — scoping it will break something" | Usage data says what it needs; the dry-run proves the shrunken policy safe before it applies. 'Might break' is what the safety net (step 4) exists to convert into 'measured.' |
| "We reviewed the humans last quarter — done" | Service accounts outnumber humans 10:1 and hold the scarier permissions; the CI role is the crown jewel most orgs never audit. Workloads first (step 1's ranking). |
| "Unused for 90 days = safe to remove" | The quarterly billing job and the DR path look unused for 89 of every 90 days. The schedule-and-runbook check per removal is one minute; the incident it prevents discredits the whole program. |
| "Deny-list the dangerous actions from the admin role" | New permissions ship into the allow-side monthly; deny-lists are permanently one release behind. Explicit allows only. |
| "Manager attests everyone's access annually — that's our review" | Rubber-stamp attestation with no usage evidence is compliance theater. Granted-vs-used data or it didn't happen. |
| "We scoped every role tightly — pen test will be clean" | The tightly-scoped role that can PassRole a broad one HAS the broad one. The escalation graph (step 6) is where scoped-everything audits still fail. |

## Red Flags

- `"Action": "*"` or `"Resource": "*"` in any reviewed policy without written justification.
- CI/CD running with admin-equivalent permissions.
- Multiple services sharing one service account.
- Standing human admin with no JIT/elevation logging.
- Access review artifacts with no usage data attached.
- PassRole/token-creation permissions never graphed.
- Last review > a year ago; wildcard count unknown.

## Verification

- [ ] Identities ranked by blast radius; review order documented.
- [ ] Granted-vs-used diff produced per reviewed identity — analyzer output linked.
- [ ] Four-classic sweep results: wildcards, convenience roles, standing admin, shared identities — each found instance dispositioned.
- [ ] Shrinks applied via dry-run→apply→watch sequence — AccessDenied monitoring linked, zero unexplained denials after a week.
- [ ] Schedule check done for every "unused" removal — noted per item.
- [ ] Escalation-graph analysis run — findings dispositioned.
- [ ] Pipeline fixes shipped: IaC-reviewed grants + paved-road roles — links; next review calendared.

## Example

Post-incident review (leaked key had admin — `secrets-incident-response` referral). Blast-radius ranking put 3 identities on top: CI deploy role (`AdministratorAccess`), the main API's service role (`s3:*`, `dynamodb:*` on `*`), and 11 humans with standing admin. Usage diff over 90 days: CI actually used 34 actions on 12 resource families → replaced with a generated policy + `iam:PassRole` locked to two named roles (killing the escalation path pmapper found); API role used `GetObject/PutObject` on two buckets and 6 DynamoDB actions on 3 tables → scoped exactly, deployed via a week of Access Analyzer dry-run, zero AccessDenied events; humans cut to 2 break-glass accounts (alarmed on use) + JIT elevation for the rest. Quarterly job trap avoided once: the "unused" `glacier:InitiateJob` belonged to the annual compliance export — found in a runbook, kept, documented. Next quarter's numbers: wildcard identities 14 → 2.

## Related skills

- `secrets-incident-response` — why blast radius keeps coming up.
- `infra-as-code-review` — the PR-time gate for new grants.
- `authn-authz-design` — the application-layer counterpart.
- `secrets-management` — identity-over-credentials, and per-service splitting.
