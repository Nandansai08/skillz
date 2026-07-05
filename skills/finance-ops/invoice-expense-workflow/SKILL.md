---
name: invoice-expense-workflow
description: >
  Use when designing or fixing how invoices and expenses flow — approval
  chains that don't bottleneck, categorization that survives audit,
  controls proportional to risk. Triggers: "expense approval process",
  "invoice workflow", "AP process is a mess", "expense policy", "audit
  trail for spend", "who approves what". NOT for evaluating what to buy
  (see vendor-evaluation) or analyzing spend after the fact (see
  budget-variance-analysis).
---

# Invoice & Expense Workflow

## Overview

Friction proportional to stakes: three signatures on a $40 lunch teaches everyone to route around the process, while the flat process lets a $30k commitment through on a distracted click. Tiered controls, separation of duties (the boring control that blocks most actual fraud), and out-of-band verification of bank-detail changes are the load-bearing walls.

## When to Use

- Building spend processes for a growing team, or unclogging ones that grew ad hoc.
- Audit prep revealed uncategorized spend, missing approvals, untraceable payments.

**When NOT to use:**
- Purchase decisions — `vendor-evaluation`.
- Post-hoc spend analysis — `budget-variance-analysis` (this workflow's clean data feeds it).

## Prerequisites

- The current state mapped honestly: how spend actually gets approved today (including the Slack-DM approvals), volumes by size band, where the pain concentrates.

## The Workflow

1. **Tier the controls by amount and risk:** e.g., <$500: auto-approved within policy, sampled retroactively; $500–$5k: one manager; $5k–$25k: manager + budget owner; >$25k or ANY new vendor/contract: procurement (`vendor-evaluation` trigger) + finance. Publish the tiers; ambiguity about "does this need approval?" is itself a cost.

2. **Design approvals around four-eyes and the real risks:** no one approves their own spend, ever, including executives; the approver must be able to JUDGE the spend (the budget owner who knows if it's sane, not a distant VP rubber-stamping); separation of duties on the money path — approve ≠ execute payment, vendor-master changes ≠ invoice approval. And the anti-BEC control: bank-detail changes verified out-of-band with a KNOWN contact — never the contact details in the email requesting the change.

3. **Kill the bottleneck mechanics explicitly:** approval SLAs (48h, then auto-ESCALATE to the approver's manager — never auto-approve, which converts vacations into control holes); delegation-with-audit for absences; batch-friendly interfaces; and the renewals calendar — auto-renewing contracts surfaced 60–90 days ahead (the surprise renewal is the most preventable bad spend in most companies, and it's a calendar entry).

4. **Make categorization happen at capture, by structure not memory:** category + department/project tags chosen at submission from a SHORT plain-language list mapped to the chart of accounts behind the scenes (everyone knows "software subscriptions"; nobody knows "GL 6240"). Receipt attached at submission (photo-at-purchase) — the receipt not attached in the moment is gone.

5. **Automate the mechanical layer, keep humans on judgment:** invoice OCR/matching, duplicate detection (same vendor+amount+date-window — duplicate payment is the most common AP error and it's a query), recurring invoices auto-routed, policy checks enforced by the tool at submission. Freed human minutes go where judgment matters: the new vendor, the unusual amount.

6. **Build the audit trail as a property, not a project:** every spend traceable request → approval (who, when) → invoice → payment → GL, in the system (not email threads); retention per jurisdiction; an exceptions log for every bypass ("emergency payment, approved verbally by X, ratified [date]") — bypasses WILL happen, and the difference between a control weakness and a finding is whether the bypass was logged and ratified. The audit-season test: any transaction's full story in five minutes.

7. **Review the workflow itself quarterly with three numbers:** approval cycle time (creeping up = bottleneck forming), exception/bypass rate (creeping up = the process losing to reality — fix it before it becomes decoration), late-payment rate (vendor leverage erodes with your payment reputation). Plus the sampled audit of the auto-approved tier — the sample is what keeps the low-friction tier low-friction.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "One approval path for everything — consistent and fair" | Uniform friction breeds workarounds at the bottom and losses at the top. Proportionality IS the control design; consistency of principle, not of process weight. |
| "Auto-approve after 48 hours — keeps things moving" | It converts every vacation into an open gate, precisely for the items someone chose not to chase. Escalate on timeout; silence is never consent on money. |
| "The exec is too senior to need approval on their own spend" | Self-approval at the top is where audits find the findings and where tone-at-the-top gets set. Four-eyes has no seniority exemption — especially not at the top. |
| "The vendor emailed new bank details — update and pay" | The compromised-mailbox 'urgent new account' is the most successful B2B fraud pattern running. Out-of-band verification with a known contact — the example's callback caught a live one. |
| "Finance can categorize everything at month-end" | Finance guessing 400 transactions' purposes produces the miscategorized GL that makes downstream analysis fiction. Capture-time categorization by the person with context, from a human-readable list. |
| "We'll handle renewals when the invoices arrive" | Arrived invoice = zero leverage, budget surprise, and `budget-variance-analysis`'s plan-error case. The 60–90-day calendar is the cheapest procurement win in existence. |

## Red Flags

- Approvals living in email threads and DMs.
- Any self-approved spend in the ledger.
- Bank-detail changes processed without callbacks.
- Duplicate payments discovered by vendors, not queries.
- Renewal invoices arriving as surprises.
- Month-end categorization archaeology.
- Bypass events with no exceptions log.

## Verification

- [ ] Tier table published; volumes per tier measured — link.
- [ ] Separation of duties demonstrated: approve/execute/vendor-master roles held by different people — role matrix shown.
- [ ] Out-of-band verification procedure documented and exercised at least once — noted.
- [ ] SLA + escalation (not auto-approve) configured — settings shown.
- [ ] Renewals calendar populated from contracts, 60–90-day alerts live — link.
- [ ] Random transaction's full trail pulled in <5 minutes — timed test done.
- [ ] Quarterly review with the three numbers scheduled; auto-tier sample audit running.

## Example

60-person company, symptoms: month-end AP chaos, two duplicate payments in a year, a $28k surprise renewal, expense approvals averaging 9 days (routed to one exec who judged nothing but delayed everything). Rebuild per the tiers: <$500 auto-within-policy (78% of transaction VOLUME — nine days of friction removed from the majority instantly, sampled at 5% monthly); mid-tier to budget owners; >$25k + new vendors to the procurement path. Separation implemented: AP runs payments, budget owners approve, vendor-master changes need finance + out-of-band verification — which, eight months later, caught a real BEC attempt (emailed "new bank details" from a compromised vendor account; the callback to the known contact killed it — the control paying for the whole project in one save). Renewals calendar built from contract ingestion: the next would-be surprise ($31k) surfaced 75 days early and got renegotiated instead. Numbers after two quarters: approval cycle 9 days → 1.4; late payments 14% → 3%; and the exceptions log's 6 entries all ratified — which is what the auditor ended up sampling, approvingly.

## Related skills

- `vendor-evaluation` — the upstream decision for the big-tier items.
- `budget-variance-analysis` — the downstream consumer of clean categorization.
- `least-privilege-review` — the same separation-of-duties instinct, systems edition.
- `secrets-incident-response` — the fraud-response cousin when controls fail.
