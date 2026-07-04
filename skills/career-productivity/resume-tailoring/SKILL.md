---
name: resume-tailoring
description: >
  Use when adapting a resume/CV to a specific job posting — mapping keywords
  honestly, rewriting bullets as measured impact (XYZ format), cutting to
  what this role's reviewer needs. Triggers: "tailor my resume for this
  job", "resume review", "improve these bullet points", "resume for this
  posting", "ATS optimization".
---

# Resume Tailoring

## When to use this skill
- A specific posting is in hand and the base resume needs targeting.
- Bullets read as duty lists ("responsible for...") and need impact conversion.
- NOT for inventing experience — tailoring means selection and framing of true things; fabrication fails at interview and is out of scope. Also not for the interview itself (`interview-prep-technical`).

## Prerequisites
- The posting's text and the candidate's full history — INCLUDING the wins that never made the resume (the tailoring conversation routinely surfaces the best material from what someone deemed "not worth mentioning").

## Workflow

1. **Decompose the posting into ranked requirements:** must-haves vs nice-to-haves vs culture signals, with the posting's own vocabulary noted (they say "data pipelines" not "ETL"; "stakeholders" not "clients"). Weight by position and repetition in the posting — the requirement in the title and the first three bullets is the screen; the laundry list at the bottom is aspiration. This map is the tailoring target; everything else serves it.

2. **Map evidence to each top requirement, honestly:** for each must-have — which real experience demonstrates it? Strong direct evidence, adjacent evidence (reframable: internal tooling work IS platform work), or a genuine gap (leave it; don't pad — one strained claim taints the credible ones, and interviewers probe exactly the strained ones). The map's gaps also tell the candidate what the cover letter or a project should address — information, not shame.

3. **Rewrite bullets in accomplished-X-measured-by-Y-through-Z shape:** action verb + outcome + measure + method — "Cut deploy time 40% (45→27 min) by parallelizing the CI test suite" beats "Responsible for CI/CD improvements." Where numbers weren't tracked, honest scale markers work: team size, request volume, user counts, frequency ("across 14 services", "for ~200 weekly users"). The banned genre: duty-restatement bullets ("worked on", "helped with", "responsible for") — every bullet answers "so what changed because you were there?"

4. **Reorder and cut for the 30-second read:** the reviewer's first pass is seconds — the top third of page one carries the match (summary line tuned to THIS role, the most relevant experience's strongest bullets first); reorder bullets within jobs so this posting's priorities lead; cut what doesn't serve this application (the irrelevant early job to one line, the proud-but-off-target project entirely — the tailored resume is a selection, and cutting is most of the tailoring). One page per ~decade of experience as the default discipline.

5. **Align vocabulary without keyword-stuffing:** use the posting's terms where they're true synonyms for what you did (their "observability" for your "monitoring" — same thing, use theirs; ATS and human screeners both match on it), spell out acronyms once where ambiguity exists, and keep the format parseable (standard section headers, no tables/columns/graphics that scramble in ATS extraction — the beautiful two-column design that arrives as word salad). Stuffing beyond truth fails at the first screen question.

6. **Run the two verification passes:** the claim audit — every bullet survivable under "tell me more about this" interrogation (numbers sourced, contribution stated at its honest size: "led" vs "contributed to" — the interview WILL find the difference); and the cold read — someone (or you, next day) reads for 30 seconds and answers "what's this person's story for THIS role?" — if the answer isn't the posting's top requirement, the top third failed (step 4 again).

7. **Keep the machinery for the next application:** a master document with EVERY bullet ever written (tailoring = selecting from it, not rewriting from scratch each time), the requirement-map habit per application, and outcome tracking (which versions got screens — a response-rate signal per framing, sample-size caveats applied). Tailoring cost per application after the master exists: ~30 minutes.

## Common pitfalls
- One resume for all postings: the generalist document that's nobody's strong match — tailoring is the difference between "qualified" and "obviously relevant," and reviewers only see the second.
- Duty bullets: a job description of the job you had, not evidence of what you did with it (step 3's conversion is the highest-value edit in the genre).
- Fabrication and inflation: the "led" that was "attended," the technology listed from one tutorial — found at interview, fatal to the credible remainder (step 6's audit).
- Keyword-stuffing past truth: the skills section as ATS bait for tools never used — passes the parser, dies in the first five minutes of the screen.
- Format vanity: multi-column layouts, graphics, headshots — scrambled by ATS extraction, and reviewers read content, not decoration (step 5).
- Burying the match: page one's top third spent on an objective statement and the oldest job — the 30-second read ends before the relevant material starts (step 4).

## Example
Candidate: backend engineer, 6 years, applying to a platform-engineering posting. Requirement decomposition: Kubernetes + IaC (title + 3 bullets = the screen), "developer experience" language throughout (culture signal), Go nice-to-have. Evidence map: strong on IaC (owned Terraform for 2 years — buried at bullet 7 of the current job), adjacent on K8s (migrated services ONTO a cluster others ran — reframed honestly as migration experience, not cluster ownership), gap on Go (left as gap; Python emphasized where the posting allowed "or similar"). Bullet conversion, the flagship: "Responsible for infrastructure automation" → "Cut new-service provisioning from 2 days to 20 minutes by building Terraform modules adopted by 6 teams" (the number surfaced from a retro doc the candidate had forgotten). Reorder: IaC bullets to position 1–2, the ML side project cut entirely. Vocabulary: their "golden paths" adopted for the internal-tooling bullet — true synonym. Cold read result: "IaC person who improves developer workflows" — the posting's exact center. Screen rate across the next 8 tailored applications: 5, versus 1-in-12 for the prior generic version. The K8s reframing held at interview precisely because it had been written at its honest size.

## Related skills
- `interview-prep-technical` — what the resume's claims must survive.
- `promotion-packet` — the internal sibling: same evidence-and-impact discipline.
- `natural-prose-editing` — de-flattening the summary and cover letter.
- `metric-definition` — the honesty standard for every number claimed.
