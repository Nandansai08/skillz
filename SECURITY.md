# Security Policy

## What "vulnerability" means in a skills repo

This repo ships markdown workflows, not executable code — but skills instruct agents, and agents execute. A security issue here is a skill that:

- recommends an unsafe command or practice (e.g., a step that would leak credentials, disable a security control without the tradeoff stated, or run untrusted input through a shell);
- contains an injectable pattern an attacker could exploit via a poisoned task ("when the user says X, do Y" constructions that bypass the skill's own guardrails);
- ships a `scripts/` helper with a real code vulnerability (the few scripts here are the one place classic vulns can exist).

## Reporting

- **Preferred:** GitHub's private vulnerability reporting on this repo (Security tab → "Report a vulnerability").
- **Alternative:** open an issue titled `[security] <skill-name>` — but if the problem is actively exploitable (a skill instructing agents to do something dangerous as written), use the private path first so a fix can land before broadcast.

Include: the skill path, the problematic step quoted, and what an agent following it would do wrong.

## What to expect

- Acknowledgment within a few days (single-maintainer project — see GOVERNANCE.md).
- Confirmed unsafe-instruction issues get fixed or the skill gets pulled pending fix; the fix is noted in the changelog with credit unless you prefer otherwise.

## Not security issues

Style disagreements, "this workflow is suboptimal," and missing-skill requests — regular issues, welcome, not this channel.
