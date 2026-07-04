---
name: human-in-the-loop-checkpoints
description: >
  Use when deciding where a long-running agent loop should pause for human
  confirmation or review — placing checkpoints by reversibility and blast
  radius, designing approvals that stay meaningful. Triggers: "when should
  the agent ask permission", "human approval step", "HITL design",
  "agent did something it shouldn't have", "too many confirmation prompts",
  "autonomous vs supervised".
---

# Human-in-the-Loop Checkpoints

## When to use this skill
- Designing an agent's autonomy boundaries: what it may do alone, what needs a human.
- Tuning an existing system that's either rubber-stamp-spamming its humans or acting beyond its mandate.
- NOT for the loop's internal structure (`agent-loop-design` — this skill designs one of its four exits) or for incident-time control (that's the kill switch, a different, blunter instrument that must ALSO exist).

## Prerequisites
- An action inventory: everything the agent CAN do (tools, permissions, side effects) — checkpoint design over an unknown action surface is security theater (`least-privilege-review`'s granted-vs-used mindset, agent edition: the best checkpoint is often a permission the agent simply doesn't have).

## Workflow

1. **Score every action on the two axes that matter: reversibility × blast radius.** Reversible + contained (edit a scratch file, run a read-only query): autonomous, always — gating these is where approval fatigue is manufactured. Irreversible OR outward-facing (send the email, merge to main, delete data, spend money, call the customer-visible API): checkpoint candidates. Irreversible AND broad (production deploy, mass communication, contract acceptance): mandatory human gate, no confidence threshold overrides it. The scoring produces the autonomy matrix — the design's actual deliverable, reviewable by security and leadership as a document (`threat-modeling` energy: the agent is an actor whose capabilities deserve the same walk).

2. **Prefer structural gates over behavioral ones:** "the agent asks before deleting" is a prompt-level hope; "the agent's credentials cannot delete" is a fact (`least-privilege-review` again — sandboxes, read-only tokens, draft-only email scopes, spend limits at the payment layer). Checkpoint the actions that MUST remain possible; remove the ones that needn't be. Every capability converted to a structural impossibility is a checkpoint you don't operate and a failure mode that can't occur.

3. **Design the checkpoint's content for a real decision, not a reflex:** the approval request carries — what the agent wants to do (the EXACT action: the diff, the email draft, the command — not a summary of it), why (the goal-chain that led here), what happens if approved (side effects, recipients, costs), the agent's own uncertainty flags, and the alternatives it considered. An approval prompt that shows "Agent wants to proceed. OK?" collects reflexes; the diff-and-reasoning version collects judgment — and the difference determines whether the human layer adds safety or latency-flavored theater.

4. **Batch and place checkpoints at phase boundaries, not per-action:** ten mid-flow interruptions produce fatigue-approval (every prompt approved unread by week two — the mechanism that converts HITL into rubber-stamp-as-a-service); the sustainable shape is checkpoint-at-milestones — plan approval before execution ("here's what I intend to change and why — proceed?"), then autonomous execution of the approved plan, then review-before-the-irreversible-commit (the PR model: agents produce reviewable artifacts, humans gate the merge). `agent-loop-design` step 5's phase structure supplies the natural seams.

5. **Define the escalation triggers beyond the static matrix — the dynamic checkpoints:** confidence collapse (the agent's own uncertainty crossing a threshold), repeated failure (the attempt ledger at N — `tool-use-loop-debugging`'s exit), constraint conflict discovered (the irreconcilable-objectives case — the agent's job becomes reporting, not choosing), and anomaly relative to the task's expected shape (a "fix the test" run proposing changes to 40 files). Each trigger hands over WITH the context package: goal, history, the specific question needing human judgment — an escalation that requires 20 minutes of reconstruction before the human can decide is an escalation that gets skipped next time.

6. **Engineer the human side as an operational system:** who approves (role, expertise-matched — the deploy checkpoint routed to someone who can read the diff), response-time expectations and what the agent does while waiting (park the run, work another task, or time out to safe-abort — NEVER auto-approve-on-timeout, the `invoice-expense-workflow` step 3 rule verbatim: timeouts escalate, silence is not consent), and the load budget: approvals-per-human-per-day tracked, because a checkpoint design that generates 200 daily approvals has designed its own bypass (`alerting-design` step 7's actionable-rate discipline — an approval queue is a pager by another name).

7. **Audit the checkpoint system with its own exhaust:** approval rate per checkpoint type (≥98% approved = the checkpoint is either mis-scoped (loosen: move it to autonomous) or unread (fix step 3's content) — investigate before deleting, the high rate has two opposite causes); override/rejection cases reviewed monthly (each rejection is either the system working or a capability to remove structurally); near-misses — approved actions later regretted — fed back as matrix tightening (`blameless-postmortem` the ones that mattered); and periodic red-team runs (does the agent respect the gates under adversarial task phrasing? do the structural limits actually hold?).

## Common pitfalls
- Uniform gating: every action prompts, humans approve by reflex within a week, and the one prompt that mattered sails through on muscle memory — fatigue is the failure mode that defeats the entire pattern (steps 1, 4).
- Behavioral gates on critical actions: "instructed not to" as the only barrier between the agent and production — prompts are preferences, permissions are facts (step 2).
- Context-free approvals: "OK to proceed?" with the actual diff a click away (never taken) — the human layer's value is exactly the judgment the summary hides (step 3).
- Auto-approve on timeout: the checkpoint that dissolves under time pressure, i.e., precisely when scrutiny mattered (step 6).
- Escalations without handoff packages: the human receives "agent stuck, help" and a 400-iteration trace — decision latency measured in archaeology (step 5; `on-call-handoff`'s state-not-just-facts rule).
- Designing for the demo's task distribution: checkpoints placed for the happy path, red-teamed never — the adversarially-phrased task that walks around the matrix is found in production instead of in the drill (step 7).

## Example
System: an agent handling customer-billing corrections (refunds, credits, plan fixes) — high stakes, high volume, and v1 gated EVERY action: 340 approvals/day, measured approval latency 4 seconds median (nobody was reading anything — rubber-stamp-as-a-service, fully operational). Redesign: action inventory scored on the matrix — lookups and draft-computations autonomous (60% of former prompts deleted outright); refunds structurally capped at $200/customer/week via the payment layer's own limits (step 2: below the cap, autonomous with daily sampled audit; the cap is a fact, not an instruction); above-cap refunds and ANY plan-contract change → checkpoint with the full package (customer history, the computed correction, the agent's reasoning, flagged anomalies); escalation triggers: confidence flag, third failed reconciliation attempt, or correction pattern anomalous vs the customer's history. Human side: routed to the billing-ops rotation (domain-matched), 2-hour SLA, timeout = park-and-queue (never auto-approve), load tracked. Results at month two: 340 → 19 approvals/day, median review time 90 seconds (they READ them now — the content redesign plus the volume drop, jointly), rejection rate 11% (the checkpoint demonstrably filtering, not decorating). The monthly audit's first catch: one "approved-then-regretted" case where the package had omitted the customer's pending dispute — the missing field added (step 7 feeding step 3), and the near-miss postmortem'd instead of buried. The red-team drill's finding: a task phrased as "test refund flow" walked past the anomaly trigger — patched by scoping the structural cap to ALL execution modes, the behavioral-gate lesson relearned cheaply.

## Related skills
- `agent-loop-design` — the escalation exit this skill designs in full.
- `least-privilege-review` — the structural-gates toolkit.
- `runaway-cost-guardrails` — the budget sibling of these safety gates.
- `invoice-expense-workflow` — the human-approval mechanics, pre-agent edition.
- `alerting-design` — the actionability discipline for the approval queue.
