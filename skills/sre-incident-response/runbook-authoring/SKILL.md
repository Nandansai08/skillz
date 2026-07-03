---
name: runbook-authoring
description: >
  Use when writing operational runbooks — step-by-step procedures a 3am
  responder can execute under stress without tribal knowledge. Triggers:
  "write a runbook for", "document this procedure", "on-call docs for this
  alert", "how do we failover", "operational playbook".
---

# Runbook Authoring

## When to use this skill
- An alert exists without a runbook, or a postmortem flagged slow mitigation.
- Capturing a procedure currently living in one person's head.
- NOT for narrative system documentation — see `docs-information-architecture`; a runbook is a procedure, not an explanation.

## Prerequisites
- The procedure actually performed at least once by someone (write from a real execution, not theory).
- A home with search + links that on-call actually uses, linked FROM the alert (`alerting-design` step 4).

## Workflow

1. **Write for the 3am reader.** Assumptions: stressed, possibly unfamiliar with this service, half-awake. Consequences: no unexplained jargon, no "simply", every command copy-pasteable with placeholders marked (`<pod-name>`), decisions as explicit branches not judgment calls. The runbook's quality bar: a competent engineer from a *different team* can execute it.

2. **Open with a header block:** what alert/symptom this covers, user impact ("checkout degraded for EU"), severity guidance, escalation contact if the runbook fails, and last-verified date. The reader decides in 10 seconds whether they're in the right document.

3. **Structure as: Verify → Mitigate → Diagnose → Escalate.**
   - **Verify** (2–3 checks): confirm the problem is real and is *this* problem — the dashboard link, the one query. False-alarm exit here.
   - **Mitigate first:** the actions that restore users (rollback, flag off, failover, drain), before any diagnosis — mirroring `incident-triage` step 4. Each action states expected effect and how long until it's visible ("error rate should drop within 2 min").
   - **Diagnose:** only after mitigation, or if mitigation options are exhausted.
   - **Escalate:** the explicit tripwire — "if not recovered after steps 1–3 or within 20 minutes, page <team> via <route>." Runbooks without escape hatches trap responders in loops.

4. **Make every step observable:** command → expected output → what-if-not. The dangerous runbook is the one where step 4 fails silently and the responder continues:
   ```
   3. Drain the node:
      kubectl drain <node> --ignore-daemonsets
      Expect: "node/<node> drained" within ~2 min.
      If pods stay Pending → capacity issue, STOP, go to Escalate.
   ```

5. **Mark destructive steps loudly.** Anything irreversible (data deletion, failover with replication lag, restart that loses state) gets a ⚠️ block stating what's at risk and any required confirmation ("check replication lag < 10s BEFORE promoting — promoting with lag loses writes"). Under stress, people execute; the warning must arrive before the command.

6. **Test it cold.** Have someone who didn't write it execute in staging (or game-day it in prod). Every place they hesitate, ask a question, or improvise is a defect — fix the doc, not the reader. An untested runbook is a rumor with formatting.

7. **Keep it alive:** last-verified date in the header, re-verified quarterly or on any architecture change (stale runbooks are worse than none — they're confidently wrong); after every real execution, a 5-minute diff pass ("what did I actually do that the doc didn't say?"); and prune runbooks for retired systems ruthlessly.

## Common pitfalls
- Explaining the architecture for three paragraphs before the first action. The 3am reader needs step 1; link the design doc for the curious.
- "Restart the service if needed" — needed when? Which command? The unexecutable step is where the runbook gets abandoned and improvisation begins.
- Automation-worthy runbooks left as prose forever. A runbook executed monthly with zero decision points is a script with extra steps — automate it and keep a one-line runbook pointing at the button. (Corollary: a runbook full of judgment calls resists automation; that's fine.)
- Screenshots of dashboards/UIs — they rot faster than text and aren't searchable. Link the live dashboard, describe the signal.
- Written by the expert, tested by the expert. The expert's tribal knowledge fills the doc's gaps invisibly; step 6 requires a non-author.

## Example
Postmortem action item: "document Redis failover" (mitigation took 25 min of improvisation). Runbook written from the incident's actual commands: verify block (2 checks distinguishing Redis-down from network partition), mitigation (promote replica — with the ⚠️ replication-lag check that the incident nearly violated), expected outputs per step, escalation tripwire at 15 min. Cold-tested by a backend dev who'd never touched Redis ops: found 3 defects (an aliased command that didn't exist on the ops box, a missing kubeconfig context switch, an ambiguous "the replica"). Next real failover: 6 minutes, executed by that same non-expert.

## Related skills
- `alerting-design` — every page links to one of these.
- `incident-triage` — the mitigate-first ordering this encodes per-service.
- `blameless-postmortem` — the source of most runbook backlog items.
