---
name: runbook-authoring
description: >
  Use when writing operational runbooks — step-by-step procedures a 3am
  responder can execute under stress without tribal knowledge. Triggers:
  "write a runbook for", "document this procedure", "on-call docs for
  this alert", "how do we failover", "operational playbook". NOT for
  narrative system documentation (see docs-information-architecture) — a
  runbook is a procedure, not an explanation.
---

# Runbook Authoring

## Overview

A runbook's reader is stressed, half-awake, and possibly from another team — every unexplained judgment call, unverifiable step, and stale command is where they stall or improvise. Write for that reader, test on that reader, and the 25-minute improvisation becomes a 6-minute procedure.

## When to Use

- An alert exists without a runbook, or a postmortem flagged slow mitigation.
- Capturing a procedure currently living in one person's head.

**When NOT to use:**
- Narrative "how the system works" docs — `docs-information-architecture`; link them, don't inline them.

## Prerequisites

- The procedure actually performed at least once by someone (write from a real execution, not theory).
- A home with search + links that on-call actually uses, linked FROM the alert (`alerting-design` step 4).

## The Workflow

1. **Write for the 3am reader.** Assumptions: stressed, possibly unfamiliar with this service, half-awake. Consequences: no unexplained jargon, no "simply", every command copy-pasteable with placeholders marked (`<pod-name>`), decisions as explicit branches not judgment calls. The quality bar: a competent engineer from a *different team* can execute it.

2. **Open with a header block:** what alert/symptom this covers, user impact ("checkout degraded for EU"), severity guidance, escalation contact if the runbook fails, and last-verified date. The reader decides in 10 seconds whether they're in the right document.

3. **Structure as: Verify → Mitigate → Diagnose → Escalate.**
   - **Verify** (2–3 checks): confirm the problem is real and is *this* problem — the dashboard link, the one query. False-alarm exit here.
   - **Mitigate first:** the actions that restore users (rollback, flag off, failover, drain), before any diagnosis — mirroring `incident-triage` step 4. Each action states expected effect and how long until visible.
   - **Diagnose:** only after mitigation, or if mitigation options are exhausted.
   - **Escalate:** the explicit tripwire — "if not recovered after steps 1–3 or within 20 minutes, page <team> via <route>." Runbooks without escape hatches trap responders in loops.

4. **Make every step observable:** command → expected output → what-if-not:
   ```
   3. Drain the node:
      kubectl drain <node> --ignore-daemonsets
      Expect: "node/<node> drained" within ~2 min.
      If pods stay Pending → capacity issue, STOP, go to Escalate.
   ```

5. **Mark destructive steps loudly.** Anything irreversible (data deletion, failover with replication lag, restart that loses state) gets a ⚠️ block stating what's at risk and any required pre-check ("check replication lag < 10s BEFORE promoting — promoting with lag loses writes"). Under stress, people execute; the warning must arrive before the command.

6. **Test it cold.** Have someone who didn't write it execute in staging (or game-day it in prod). Every place they hesitate, ask a question, or improvise is a defect — fix the doc, not the reader. The author can't run this test: their tribal knowledge fills the gaps invisibly.

7. **Keep it alive:** last-verified date in the header, re-verified quarterly or on any architecture change (stale runbooks are worse than none — they're confidently wrong); after every real execution, a 5-minute diff pass ("what did I actually do that the doc didn't say?"); prune runbooks for retired systems. And automate the automatable: a runbook executed monthly with zero decision points is a script with extra steps — automate it, leave a one-line runbook pointing at the button.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "The team knows this system — no need to spell out the basics" | The 3am responder is whoever's on rotation, increasingly from adjacent teams. The doc written for insiders is the doc that fails exactly when coverage is thinnest. |
| "'Restart the service if needed' — they'll know when" | Needed when? Which command? Which instance first? Every judgment call left implicit is where execution stalls and improvisation starts — and improvisation at 3am wrote half the postmortem archive. |
| "I wrote it, I tested it, it works" | The author's tribal knowledge silently fills every gap in their own doc. Only the non-author cold test (step 6) measures what's actually on the page. |
| "Screenshots make it clearer than text" | Screenshots rot faster than any prose and can't be searched or copy-pasted. Link the live dashboard; describe the signal in words. |
| "It's documented — that's what matters. Verification dates are bureaucracy" | An unverified runbook after an architecture change is confidently wrong — worse than absent, because the reader trusts it. The date is the reader's freshness signal. |
| "We'll write the runbook after the next incident teaches us more" | The next incident IS the cold test, at production prices. Write from the last execution now; the diff pass keeps it honest. |

## Red Flags

- Alert firing with no runbook link attached.
- Runbooks opening with three paragraphs of architecture before the first action.
- Commands that aren't copy-pasteable (prose descriptions of what to type).
- No escalation tripwire — the doc's failure mode is an infinite loop.
- Destructive steps without pre-checks or warnings.
- Last-verified dates absent or predating the last architecture change.
- A "runbook" executed monthly with zero decisions that nobody has automated.

## Verification

- [ ] Header block complete: symptom, impact, severity, escalation, last-verified date.
- [ ] Verify→Mitigate→Diagnose→Escalate structure with an explicit tripwire (time or step count).
- [ ] Every step has command + expected output + what-if-not — spot-check three.
- [ ] Destructive steps carry ⚠️ + pre-checks — grep for the irreversible commands and check.
- [ ] Cold-tested by a non-author; their stuck-points fixed — test date and findings noted.
- [ ] Linked from the alert that leads here — alert config shown.

## Example

Postmortem action item: "document Redis failover" (mitigation took 25 min of improvisation). Runbook written from the incident's actual commands: verify block (2 checks distinguishing Redis-down from network partition), mitigation (promote replica — with the ⚠️ replication-lag check that the incident nearly violated), expected outputs per step, escalation tripwire at 15 min. Cold-tested by a backend dev who'd never touched Redis ops: found 3 defects (an aliased command that didn't exist on the ops box, a missing kubeconfig context switch, an ambiguous "the replica"). Next real failover: 6 minutes, executed by that same non-expert.

## Related skills

- `alerting-design` — every page links to one of these.
- `incident-triage` — the mitigate-first ordering this encodes per-service.
- `blameless-postmortem` — the source of most runbook backlog items.
