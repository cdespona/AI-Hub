---
name: feedback-360
description: "Conduct concise, role-aware 360-feedback interviews and draft or revise Markdown feedback files. Use when the user wants to prepare teammate feedback, a self-evaluation, a feedback cycle, or PROs/CONs in a structured feedback folder."
---

# Feedback 360

Create concise, evidence-based feedback that preserves the user's voice and the existing Markdown format.

## Workflow

1. Confirm the target person, role, evaluation period, and folder. Treat `human/` as user-owned: edit only files the user explicitly puts in scope.
2. Read the target Markdown file and, if linked, the previous feedback. Preserve frontmatter, historical links, headings, and unrelated sections.
3. Ask one brief, role-aware interview. Adapt the prompts to the role:
   - **Engineer:** technical delivery, autonomy, quality, collaboration, and growth.
   - **Tech Lead / EM:** team direction, prioritization, cross-team coordination, people leadership, and leverage.
   - **Product Manager:** priorities, context, outcomes, risks, stakeholders, and engineering partnership.
4. If information is missing, ask only the next useful question. Do not invent examples or overstate second-hand impressions.
5. Draft the file after receiving the answers. Use concrete, respectful PROs and actionable CONs that describe growth opportunities rather than failures.
6. Re-read the finished file and report its exact path.

## Self-evaluations

When the target is the user, distinguish self-review from feedback received:

- Write only the requested self-evaluation sections, normally `## PROs` and `## CONs`.
- Treat a prior `## Feedback` section as historical teammate input. Preserve it; do not present it as the user's own assessment or overwrite it unless explicitly requested.

## Writing rules

- Match the existing language, voice, headings, and list style.
- Base each statement on the user's input. Label limited visibility or second-hand information cautiously, or omit it if it is not reliable enough.
- Include an example only when the user supplied one.
- Make CONs specific and forward-looking: describe the behavior, why it matters, and a practical next move.
- Credit shared work accurately; do not assign team outcomes to one person alone.
- Avoid generic praise, performance ratings, diagnoses, and claims about intent.
