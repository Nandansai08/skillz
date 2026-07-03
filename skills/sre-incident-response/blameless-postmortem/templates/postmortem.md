# Postmortem: <one-line incident summary>

**Date:** YYYY-MM-DD | **Severity:** SEV_ | **Status:** Draft / Reviewed / Actions complete
**Authors:** <roles, not blame-targets> | **Review meeting:** YYYY-MM-DD

## Impact
- Duration (user-facing): __ min (HH:MM–HH:MM UTC)
- Users/requests affected: __
- Business cost (revenue / SLA / error budget): __
- Detection gap (impact start → first human awareness): __ min

## Summary
<3–5 sentences: what broke, what users saw, how it was mitigated.>

## Timeline (UTC)
| Time | Event / decision | Source |
|------|------------------|--------|
| HH:MM | <change that armed the trap — may predate the incident> | deploy log |
| HH:MM | impact begins | metric |
| HH:MM | first alert / first human awareness | alert |
| HH:MM | <decision point: considered X, chose Y because Z> | channel |
| HH:MM | mitigation applied | |
| HH:MM | user-facing recovery confirmed | metric |

## Contributing factors
<3–6. For each: what it was, and why the system permitted it. "Human error" is not a factor — dig one level deeper.>
1. **Trigger:**
2. **Why defenses missed it:**
3. **Why detection was slow:**
4. **Why mitigation was slow:**
5. **What amplified impact:**

## What went well
<Detection, tooling, or decisions worth reinforcing.>

## Action items
| # | Action | Type (prevent/detect/mitigate) | Owner | Ticket | Due |
|---|--------|-------------------------------|-------|--------|-----|
| 1 | | | | | |

## Where else does this pattern exist?
<The class-sweep: sibling services, same config system, same gap.>
