---
name: resume-tailoring
description: >
  Use when adapting a resume/CV to a specific job posting — mapping
  keywords honestly, rewriting bullets as measured impact, cutting to
  what this role's reviewer needs. Triggers: "tailor my resume for this
  job", "resume review", "improve these bullet points", "resume for this
  posting", "ATS optimization". NOT for inventing experience — tailoring
  is selection and framing of true things; fabrication fails at interview
  and is out of scope.
---

# Resume Tailoring

## Overview

Tailoring is selection: decompose the posting into ranked requirements, map true evidence to each, and rebuild the top third of page one around the match — because the reviewer's first pass is thirty seconds. Every bullet answers "so what changed because you were there?", and every claim survives "tell me more about this."

## When to Use

- A specific posting is in hand and the base resume needs targeting.
- Bullets read as duty lists ("responsible for...") and need impact conversion.

**When NOT to use:**
- Inventing or inflating experience — fails at interview, taints the credible remainder, out of scope.
- The interview itself — `interview-prep-technical`.

## Prerequisites

- The posting's text and the candidate's full history — INCLUDING wins that never made the resume (tailoring routinely surfaces the best material from what someone deemed "not worth mentioning").

## The Workflow

1. **Decompose the posting into ranked requirements:** must-haves vs nice-to-haves vs culture signals, with the posting's own vocabulary noted (they say "data pipelines" not "ETL"). Weight by position and repetition — the requirement in the title and first three bullets is the screen; the bottom laundry list is aspiration.

2. **Map evidence to each top requirement, honestly:** strong direct evidence, adjacent evidence (reframable: internal tooling work IS platform work), or a genuine gap (leave it; don't pad — one strained claim taints the credible ones, and interviewers probe exactly the strained ones). The gaps also tell you what the cover letter or a project should address — information, not shame.

3. **Rewrite bullets in accomplished-X-measured-by-Y-through-Z shape:** action verb + outcome + measure + method — "Cut deploy time 40% (45→27 min) by parallelizing the CI test suite" beats "Responsible for CI/CD improvements." Where numbers weren't tracked, honest scale markers work: team size, request volume, frequency ("across 14 services"). The banned genre: duty-restatement ("worked on", "helped with") — every bullet answers "so what changed because you were there?"

4. **Reorder and cut for the 30-second read:** the top third of page one carries the match (summary tuned to THIS role, the most relevant experience's strongest bullets first); bullets within jobs reordered so this posting's priorities lead; cut what doesn't serve this application (the irrelevant early job to one line, the proud-but-off-target project entirely — cutting IS most of the tailoring). One page per ~decade as the default.

5. **Align vocabulary without keyword-stuffing:** use the posting's terms where they're true synonyms for what you did (their "observability" for your "monitoring" — ATS and humans both match on it); acronyms spelled out once; format parseable (standard headers, no tables/columns/graphics that scramble in ATS extraction). Stuffing beyond truth fails at the first screen question.

6. **Run the two verification passes:** the claim audit — every bullet survivable under "tell me more" (numbers sourced, contribution at its honest size: "led" vs "contributed to" — the interview WILL find the difference); and the cold read — someone reads for 30 seconds and answers "what's this person's story for THIS role?" If the answer isn't the posting's top requirement, the top third failed.

7. **Keep the machinery for the next application:** a master document with EVERY bullet ever written (tailoring = selecting from it, not rewriting from scratch), the requirement-map habit per application, and outcome tracking (which versions got screens). Cost per application once the master exists: ~30 minutes.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "One strong general resume works for everything" | The generalist document is nobody's obvious match, and reviewers only shortlist obvious. The example's screen rate: 1-in-12 generic vs 5-in-8 tailored. |
| "Listing my responsibilities shows what I did" | It shows what the JOB was. The reviewer is buying what changed because you were there — the XYZ conversion is the single highest-value edit in the genre. |
| "Stretch 'contributed to' into 'led' — everyone does it" | And interviewers probe exactly the stretched claims, whose collapse taints every honest one on the page. Precision in both directions; adjacency reframing is honest, inflation isn't. |
| "Cram the keywords — beat the ATS first" | Stuffing past truth passes the parser and dies in the first five minutes of the screen. True-synonym alignment (their word for your real work) serves both readers. |
| "The two-column design makes me stand out" | It stands out as word salad after ATS extraction. Content parseable first; distinctiveness lives in the bullets' numbers, not the layout. |
| "No metrics were tracked for my work — nothing to quantify" | Scale markers are metrics: team size, volume, frequency, blast radius. And the tailoring conversation routinely excavates numbers from old retro docs (the example's provisioning stat was in one). |

## Red Flags

- The same resume attached to every application.
- Bullets starting with "Responsible for" or "Helped with."
- Skills sections listing tools used once in a tutorial.
- The top third of page one spent on an objective statement and the oldest job.
- Claims the candidate visibly can't expand on when asked.
- No master document; every application a from-scratch rewrite.

## Verification

- [ ] Requirement map built for the posting (must/nice/culture, weighted) — attached.
- [ ] Every top requirement mapped to evidence or honestly gapped — no padding.
- [ ] Bullets pass the so-what test in XYZ shape — spot-check five.
- [ ] Cold read returns the posting's top requirement as "the story" — tester noted.
- [ ] Claim audit passed: each bullet expandable under interrogation.
- [ ] Master document updated with any newly-excavated bullets.

## Example

Candidate: backend engineer, 6 years, applying to a platform-engineering posting. Requirement decomposition: Kubernetes + IaC (title + 3 bullets = the screen), "developer experience" language throughout (culture signal), Go nice-to-have. Evidence map: strong on IaC (owned Terraform for 2 years — buried at bullet 7), adjacent on K8s (migrated services ONTO a cluster others ran — reframed honestly as migration experience, not cluster ownership), gap on Go (left as gap). Bullet conversion, the flagship: "Responsible for infrastructure automation" → "Cut new-service provisioning from 2 days to 20 minutes by building Terraform modules adopted by 6 teams" (the number surfaced from a forgotten retro doc). Reorder: IaC bullets to positions 1–2, the ML side project cut. Vocabulary: their "golden paths" adopted for the internal-tooling bullet — true synonym. Cold read: "IaC person who improves developer workflows" — the posting's exact center. Screen rate across the next 8 tailored applications: 5, vs 1-in-12 for the prior generic version. The K8s reframing held at interview precisely because it had been written at its honest size.

## Related skills

- `interview-prep-technical` — what the resume's claims must survive.
- `promotion-packet` — the internal sibling: same evidence-and-impact discipline.
- `natural-prose-editing` — de-flattening the summary and cover letter.
- `metric-definition` — the honesty standard for every number claimed.
