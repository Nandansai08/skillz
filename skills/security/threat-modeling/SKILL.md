---
name: threat-modeling
description: >
  Use when assessing what could go wrong security-wise in a feature or
  system design — a lightweight STRIDE pass over assets, actors, and
  attack surface, producing ranked mitigations. Triggers: "threat model
  this", "security review of this design", "what could an attacker do",
  "is this design secure", "security implications of this feature". NOT
  for reviewing written code (see secure-code-review) — this happens on
  the diagram, not the diff.
---

# Threat Modeling

## Overview

Threats concentrate where trust levels change; a 90-minute STRIDE walk over the boundaries, at design time, finds the authz gap and the webhook-trust mistake while they cost a diagram edit instead of an incident. The deliverable is ranked threats with owners — not a list of scary sentences.

## When to Use

- A new feature/system touches auth, money, PII, or a new external surface — model at design time, before code.
- Revisiting an existing system after an architecture change.

**When NOT to use:**
- Code-level vulnerability hunting — `secure-code-review`.

## Prerequisites

- A data-flow sketch of the feature: components, data stores, external parties, and the arrows between them. Ten boxes max — model at the feature's altitude, not the whole company's.

## The Workflow

1. **Draw trust boundaries onto the sketch first.** A boundary is anywhere the level of trust changes: internet→edge, app→database, your service→vendor API, user browser→backend, tenant→tenant inside shared infra, and service→service (the VPC is not a trust level — SSRF and compromised pods dissolve it). Threats concentrate at crossings; the diagram's purpose is making crossings visible.

2. **Inventory assets and actors in one list each.** Assets: what an attacker wants (credentials, PII, payment data, compute, your users' trust via your domain). Actors: anonymous internet, authenticated user, *malicious authenticated user* (the most under-modeled one), tenant-peer, insider, compromised dependency.

3. **Walk STRIDE per boundary crossing** — for each arrow crossing a boundary, ask which apply:
   - **S**poofing: can the caller lie about who they are? (auth on this hop?)
   - **T**ampering: can the data be modified in flight/at rest? (integrity, signatures)
   - **R**epudiation: could an actor deny an action? (audit logging)
   - **I**nformation disclosure: what leaks if this hop is observed/errored? (encryption, error verbosity, logs)
   - **D**enial of service: what happens at 1000× volume on this arrow? (limits, quotas)
   - **E**levation of privilege: can this input make the callee do something the caller couldn't? (injection/deserialization/confused-deputy)
   Write threats as attack sentences: "A malicious authenticated user edits the `user_id` in the webhook replay body to read another tenant's invoices."

4. **Rank by likelihood × impact, crudely but honestly.** High/medium/low each axis. Internet-reachable + unauthenticated + valuable asset = top of list. Resist the fun exotic threat outranking the boring critical one (IDOR beats Bluetooth exfiltration in almost every web app's model).

5. **Assign each top threat one of four dispositions:** mitigate (name the control: authz check, signature verification, rate limit), eliminate (remove the surface — often the best control is not storing/collecting the data at all), transfer (vendor contract, insurance), or accept (written down, signed by someone entitled to accept it). "Accept by silence" is the disposition to hunt down.

6. **Turn mitigations into tracked work:** tickets with the threat sentence as context, acceptance criteria naming the test that proves the control ("integration test: tenant A requesting tenant B's invoice gets 404"). A threat model whose output isn't in the backlog was a workshop, not a model. Remember the model's own exhaust: the audit logs and error messages added as mitigations are themselves disclosure surfaces — walk them too.

7. **Timebox and revisit.** 60–90 minutes gets 80% of the value for a feature-sized model. Re-run when the diagram changes (new external integration, new data class, auth model change) — staleness is the main failure mode of threat models that exist.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "We don't have time for a security workshop this sprint" | Feature-sized modeling is 90 minutes, and it's the cheapest hour security ever costs — the same findings post-launch arrive as incidents with disclosure timelines. |
| "Our users are businesses — they won't attack us" | The malicious AUTHENTICATED user is the top real-world web attacker: IDOR, privilege escalation, tenant crossing. Trusting logged-in users is the most expensive courtesy in security. |
| "It's behind the VPN/VPC, so it's trusted" | One SSRF or compromised pod gives an attacker an internal HTTP client, and the 'trusted' network becomes their toolkit. Service-to-service hops are boundaries. |
| "The vendor is under contract — their webhook is trusted input" | Contracts don't authenticate packets. Unsigned webhooks are anonymous internet input wearing a vendor's name; signature verification is the actual trust. |
| "We'll model the whole platform properly next quarter" | The 60-box, 4-hour session exhausts everyone and ranks nothing. Feature-sized bites now beat the comprehensive model that never converges. |
| "We listed the threats — job done" | A threat list with no dispositions and no tickets changed nothing. Steps 5–6 are the deliverable; the list is its appendix. |

## Red Flags

- The model contains only anonymous-attacker threats.
- Threats written as categories ("injection risk") instead of attack sentences.
- No disposition column; scary sentences with no owners.
- The exotic threat ranked above the boring IDOR.
- Accepted risks that nobody with authority actually signed.
- The feature shipped with new external arrows the model never saw.

## Verification

- [ ] Data-flow sketch with trust boundaries drawn — attached to the model doc.
- [ ] Actor list includes the malicious authenticated user and tenant-peer.
- [ ] Threats written as attack sentences — spot-check three.
- [ ] Every top-ranked threat has a disposition; accepts carry a named signer.
- [ ] Mitigations ticketed with proving-test acceptance criteria — links.
- [ ] Revisit trigger stated (what diagram change re-runs the model).

## Example

Feature: "customers can share read-only invoice links." Sketch: browser → share-service → invoice-store, plus emailed link. Boundary walk found: link tokens guessable if sequential (Spoofing/IDOR — mitigate: 128-bit random tokens); tokens in email = tokens in forwarded email (Info disclosure — accept for v1, *written*, plus expiry mitigation: 30-day TTL); no revocation (Tampering with intent — mitigate: revoke endpoint); token brute-force (DoS/E — rate limit per IP + constant-time compare); shared invoice reveals other line items of a multi-tenant PDF renderer bug (found by asking "what does the renderer trust?"). Five tickets, one written acceptance, and the tenant-isolation test became the PR's acceptance criterion. 75 minutes.

## Related skills

- `secure-code-review` — verifying the mitigations in the implementation.
- `authn-authz-design` — deep dive when the S and E rows dominate.
- `least-privilege-review` — the standing control for confused-deputy threats.
- `input-validation-boundaries` — where the T and E mitigations get placed.
