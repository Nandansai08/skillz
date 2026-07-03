---
name: invoice-expense-workflow
description: >
  Use when designing or fixing how invoices and expenses flow — approval
  chains that don't bottleneck, categorization that survives audit,
  controls proportional to risk. Triggers: "expense approval process",
  "invoice workflow", "AP process is a mess", "expense policy", "audit
  trail for spend", "who approves what".
---

# Invoice & Expense Workflow

## When to use this skill
- Building spend processes for a growing team, or unclogging ones that grew ad hoc.
- Audit prep revealed uncategorized spend, missing approvals, or untraceable payments.
- NOT for evaluating what to buy (`vendor-evaluation`) or analyzing the spend after the fact (`budget-variance-analysis` — which this workflow's clean data feeds).

## Prerequisites
- The current state mapped honestly: how spend actually gets approved today (including the Slack-DM approvals and the CEO's card), volumes by size band, and where the pain concentrates (late payments? surprise renewals? month-end chaos?).

## Workflow

1. **Tier the controls by amount and risk — friction proportional to stakes:** e.g., <$500: auto-approved within policy, sampled retroactively; $500–$5k: one manager approval; $5k–$25k: manager + budget owner; >$25k or ANY new vendor/contract: procurement review (`vendor-evaluation` trigger) + finance. Uniform friction is the classic failure in both directions — three signatures on a $40 expense teaches everyone to hate and route around the process, while the same flat process lets a $30k commitment through on one distracted click. Publish the tiers; ambiguity about "does this need approval?" is itself a cost.

2. **Design approvals around the four-eyes minimum and the real risks:** no one approves their own spend, ever, including (especially) executives; the approver must be someone who can JUDGE the spend (the budget owner who knows if it's sane, not a hierarchy-distant VP rubber-stamping a queue); separation of duties on the money path — whoever approves doesn't also execute payment, whoever adds vendors doesn't also approve invoices (the boring control that blocks most actual fraud, including the compromised-email "urgent wire" attack — which also demands: bank-detail changes verified out-of-band with the vendor via a known contact, never the contact details in the email requesting the change).

3. **Kill the bottleneck mechanics explicitly:** approval SLAs (48h, then auto-escalate to the approver's manager — not auto-approve, which converts vacations into control holes); delegation-with-audit for absences; batch-friendly interfaces (the approver facing 40 one-click items with context shown beats 40 emails); and the renewals calendar — auto-renewing contracts surfaced 60–90 days ahead (the surprise renewal is the most preventable bad spend in most companies, and it's a calendar entry — `budget-variance-analysis`'s plan-error case, prevented upstream).

4. **Make categorization happen at capture, by structure not memory:** category (+ department/project tags) chosen at submission while context exists, from a SHORT, plain-language list mapped to the chart of accounts behind the scenes (nobody outside finance knows what "GL 6240" means; everyone knows "software subscriptions") — month-end recategorization by finance archaeology is the alternative, and it's both expensive and wrong. Receipt/documentation required at submission (photo-at-purchase in the tool), because the receipt that isn't attached in the moment is gone (step 6's audit trail starts here).

5. **Automate the mechanical layer, keep humans on judgment:** invoice ingestion (OCR/e-invoice) matched against PO or expected-spend where volumes justify; duplicate detection (same vendor+amount+date-window — duplicate payment is the most common AP error and it's a query); recurring known invoices auto-routed; policy checks (limits, categories, missing receipts) enforced by the tool at submission, not by human review after. The human minutes freed get spent where judgment matters: the new vendor, the unusual amount, the contract.

6. **Build the audit trail as a property, not a project:** every spend traceable as request → approval (who, when) → invoice/receipt → payment → GL entry, in the system (not in email threads); documentation retention per your jurisdiction's rules (typically 7 years); an exceptions log for every process bypass ("emergency payment, approved verbally by X, ratified [date]") — because bypasses WILL happen, and the difference between a control weakness and a finding is whether the bypass was logged and ratified. When audit season arrives, the test is pulling any transaction's full story in five minutes.

7. **Review the workflow itself quarterly with three numbers:** approval cycle time (creeping up = bottleneck forming), exception/bypass rate (creeping up = the process is losing to reality — fix the process before it becomes decoration), and late-payment rate (vendor relationships and late fees both — `vendor-evaluation`'s leverage erodes with your payment reputation). Plus the sampled-audit of the auto-approved tier: the low-friction tier stays low-friction BECAUSE the sample catches abuse cheaply.

## Common pitfalls
- Friction uniformity: the $40 lunch and the $40k contract on the same approval path — one direction breeds workarounds, the other breeds losses (step 1's tiers).
- Auto-approve on timeout: the SLA "solved" by converting every slow approver into an open gate. Escalate, never auto-approve (step 3).
- The email-thread audit trail: approvals live in inboxes, the approver left the company, the auditor asks for evidence — reconstruction archaeology at $400/hour (step 6's in-system property).
- Bank-detail changes on request: the "updated invoice" with new account details, paid — the out-of-band verification (step 2) is the entire defense against the most successful B2B fraud pattern running.
- Categorization at month-end: finance guessing what 400 transactions were, producing the miscategorized GL that makes `budget-variance-analysis` fiction downstream (step 4).
- Process-by-accretion: each incident adds a rule, no rule ever retires, and year three's flowchart requires its own onboarding — the quarterly review (step 7) prunes as well as patches.

## Example
60-person company, symptoms: month-end AP chaos, two duplicate payments in a year, a $28k surprise renewal, and expense approvals averaging 9 days (routed to one exec who judged nothing but delayed everything). Rebuild per the tiers: <$500 auto-within-policy (78% of transaction VOLUME — nine days of friction removed from the majority instantly, sampled at 5% monthly); mid-tier to budget owners (who actually know); >$25k + new vendors to the procurement path. Separation implemented: AP runs payments, budget owners approve, vendor-master changes need finance + out-of-band verification — which, eight months later, caught a real BEC attempt (emailed "new bank details" from a compromised vendor account; the callback to the known contact killed it — the control paying for the whole project in one save). Renewals calendar built from contract ingestion: the next would-be surprise ($31k) surfaced 75 days early and got renegotiated instead (`vendor-evaluation`). Duplicate detection: one catch in the first quarter. Numbers after two quarters: approval cycle 9 days → 1.4; late payments 14% → 3%; month-end close two days shorter — and the exceptions log's 6 entries all ratified, which is what the auditor ended up sampling, approvingly.

## Related skills
- `vendor-evaluation` — the upstream decision for the big-tier items.
- `budget-variance-analysis` — the downstream consumer of clean categorization.
- `least-privilege-review` — the same separation-of-duties instinct, systems edition.
- `secrets-incident-response` — the BEC/fraud response cousin when controls fail.
