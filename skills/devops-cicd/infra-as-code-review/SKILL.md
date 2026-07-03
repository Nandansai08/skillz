---
name: infra-as-code-review
description: >
  Use when reviewing Terraform/OpenTofu/CloudFormation/Pulumi changes —
  reading plans, judging blast radius, catching destroy-and-recreate,
  state and drift issues. Triggers: "review this terraform", "terraform plan
  output", "is this infra change safe", "why does it want to destroy",
  "IaC review", "cloudformation changeset".
---

# Infra-as-Code Review

## When to use this skill
- Reviewing an IaC pull request or a `terraform plan` before apply.
- Investigating why a plan wants to change/destroy something unexpected.
- NOT for designing the infrastructure itself — this is the review/safety pass on changes.

## Prerequisites
- The rendered plan/changeset for the *target environment* — never review IaC from the diff alone; the `.tf` diff shows intent, the plan shows consequences.

## Workflow

1. **Read the plan summary line first, then every destroy.** `Plan: 3 to add, 1 to change, 2 to destroy.` Each destroy/replace gets individually justified. The killer pattern is **replace** (`-/+` / `must be replaced`) on stateful resources — databases, volumes, queues — where "replace" means "delete your data, create an empty one."

2. **Trace every replace to its forcing attribute.** The plan marks it: `# forces replacement`. Common accidental forcers: renaming a resource block (that's a delete+create — use `moved {}` blocks / `terraform state mv`), changing an immutable attribute (AZ, engine version downgrade, name fields), list-order changes on order-sensitive attributes. If the replacement is truly intended on a stateful resource, the PR needs a data-migration plan attached, not just approval.

3. **Judge blast radius by environment and dependency fan-out.** Which state/workspace does this apply to? A module change ripples into every environment that consumes it — check module versioning (pinned tags per env) so prod doesn't take the change the moment it merges. Shared resources (VPC, IAM, DNS) fail bigger than app resources; review them at higher scrutiny.

4. **Security scan the diff:** new IAM policies for wildcards (`"Action": "*"`, `"Resource": "*"`), security-group rules opening `0.0.0.0/0` on non-web ports, disabled encryption flags, public ACLs on buckets, secrets as literals in config (see `secrets-management` for Terraform-state exposure). `tfsec`/`checkov`/`trivy config` automate the floor in CI; review still owns the contextual calls.

5. **Check for drift traps.** Plan shows changes nobody's PR made → the resource was hand-edited in the console. Decide explicitly: adopt the manual change into code, or let the apply revert it — silently reverting someone's emergency console fix at deploy time is a classic secondary incident. `terraform plan -refresh-only` isolates pure drift from PR changes.

6. **Review the lifecycle guards on stateful resources.** `prevent_destroy = true` on databases/state buckets, `deletion_protection` on RDS, retain policies on CloudFormation. These belong in the code *before* the near-miss; asking for them in review is cheap.

7. **Apply discipline:** plan artifact from CI is what gets applied (`terraform apply plan.tfplan` — not a fresh plan that may differ), applies run through the pipeline with state locking, never from laptops, and out-of-band applies show up in state-lock/audit logs.

## Common pitfalls
- Approving from the `.tf` diff because it "looks small." A one-line attribute change can be a fleet replacement; only the plan knows.
- `count`/index-based resources reordered: removing item 0 of a `count` list renames every subsequent resource → mass replace. Prefer `for_each` keyed on stable names; migrate with `moved` blocks.
- Module bump "to get one fix" pulling a hidden major with new defaults. Read the module changelog like any dependency (`dependency-upgrade` applies).
- Reviewing staging's plan, applying to prod, assuming equivalence. Different state = different plan; render the plan per environment.
- Treating `terraform plan` clean output as "no impact" for changes touching providers/versions — provider upgrades can alter behavior at *apply* time. Soak provider bumps in lower environments first.

## Example
PR: "rename redis module + bump instance size." Plan on staging: `1 to add, 0 to change, 1 to destroy` — the rename forced replace on the ElastiCache cluster, flushing the session store. Review outcome: `moved {}` block added (plan becomes `1 to change` — in-place resize only), `prevent_destroy` added to the cluster, and a follow-up ticket for the drift the refresh-only plan surfaced (a console-added security-group rule from an incident two weeks prior — adopted into code).

## Related skills
- `deployment-rollback-plan` — expand/contract thinking for stateful infra changes.
- `secrets-management` — Terraform state and literal-credential handling.
- `least-privilege-review` — deep pass on the IAM policies step 4 flags.
