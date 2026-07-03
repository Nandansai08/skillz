---
name: threat-modeling
description: >
  Use when assessing what could go wrong security-wise in a feature or system
  design — a lightweight STRIDE pass over assets, actors, and attack surface,
  producing ranked mitigations. Triggers: "threat model this", "security
  review of this design", "what could an attacker do", "is this design
  secure", "security implications of this feature".
---

# Threat Modeling

## When to use this skill
- A new feature/system touches auth, money, PII, or a new external surface — model at design time, before code.
- Revisiting an existing system after an architecture change.
- NOT for reviewing written code — that's `secure-code-review`; this happens on the diagram, not the diff.

## Prerequisites
- A data-flow sketch of the feature: components, data stores, external parties, and the arrows between them. Ten boxes max — model at the feature's altitude, not the whole company's.

## Workflow

1. **Draw trust boundaries onto the sketch first.** A boundary is anywhere the level of trust changes: internet→edge, app→database, your service→vendor API, user browser→backend, tenant→tenant inside shared infra. Threats concentrate at boundary crossings; the diagram's purpose is making crossings visible.

2. **Inventory assets and actors in one list each.** Assets: what an attacker wants (credentials, PII, payment data, compute, your users' trust via your domain). Actors: anonymous internet, authenticated user, *malicious authenticated user* (the most under-modeled one), tenant-peer, insider, compromised dependency.

3. **Walk STRIDE per boundary crossing** — for each arrow crossing a boundary, ask which apply:
   - **S**poofing: can the caller lie about who they are? (auth on this hop?)
   - **T**ampering: can the data be modified in flight/at rest? (integrity, signatures)
   - **R**epudiation: could an actor deny an action? (audit logging)
   - **I**nformation disclosure: what leaks if this hop is observed/errored? (encryption, error verbosity, logs)
   - **D**enial of service: what happens at 1000× volume on this arrow? (limits, quotas)
   - **E**levation of privilege: can this input make the callee do something the caller couldn't? (the injection/deserialization/confused-deputy family)
   Write threats as attack sentences: "A malicious authenticated user edits the `user_id` in the webhook replay body to read another tenant's invoices."

4. **Rank by likelihood × impact, crudely but honestly.** High/medium/low each axis. Internet-reachable + unauthenticated + valuable asset = top of list. Resist the fun exotic threat outranking the boring critical one (IDOR beats Bluetooth-based exfiltration in almost every web app's model).

5. **Assign each top threat one of four dispositions:** mitigate (name the control: authz check, signature verification, rate limit), eliminate (remove the surface — often the best control is not storing/collecting the data at all), transfer (vendor contract, insurance), or accept (written down, signed by someone entitled to accept it). "Accept by silence" is the disposition to hunt down.

6. **Turn mitigations into tracked work:** tickets with the threat sentence as context, acceptance criteria naming the test that proves the control ("integration test: tenant A requesting tenant B's invoice gets 404"). A threat model whose output isn't in the backlog was a workshop, not a model.

7. **Timebox and revisit.** 60–90 minutes gets 80% of the value for a feature-sized model. Re-run when the diagram changes (new external integration, new data class, auth model change) — staleness is the main failure mode of threat models that exist.

## Common pitfalls
- Modeling the whole platform in one session — 60 boxes, 4 hours, everyone exhausted, nothing ranked. Feature-sized bites.
- Only modeling the anonymous attacker. Logged-in users attacking authz (IDOR, privilege escalation between roles, tenant crossing) is where most real web findings live.
- Producing threats without dispositions — a list of scary sentences with no owner changes nothing (step 5/6 are the deliverable).
- Trusting the *internal* network: "it's behind the VPC" as a universal mitigation dissolves on the first SSRF or compromised pod. Boundaries include service-to-service.
- Treating the vendor/webhook/third-party arrows as trusted because there's a contract. Signature-verify webhooks; scope vendor tokens (`least-privilege-review`).
- Forgetting the model's own exhaust: the audit logs and error messages added as mitigations are themselves information-disclosure surfaces.

## Example
Feature: "customers can share read-only invoice links." Sketch: browser → share-service → invoice-store, plus emailed link. Boundary walk found: link tokens guessable if sequential (Spoofing/IDOR — mitigate: 128-bit random tokens); tokens in email = tokens in forwarded email (Info disclosure — accept for v1, *written*, plus expiry mitigation: 30-day TTL); no revocation (Tampering with intent — mitigate: revoke endpoint); token brute-force (DoS/E — rate limit per IP + constant-time compare); shared invoice reveals other line items of a multi-tenant PDF renderer bug (found by asking "what does the renderer trust?"). Five tickets, one written acceptance, and the tenant-isolation test became the PR's acceptance criterion. 75 minutes.

## Related skills
- `secure-code-review` — verifying the mitigations in the implementation.
- `authn-authz-design` — deep dive when the S and E rows dominate.
- `least-privilege-review` — the standing control for confused-deputy threats.
- `input-validation-boundaries` — where the T and E mitigations get placed.
