---
name: write-spec
description: Turn an accepted intent.md into a requirements and design spec
  (spec.md) for product owner review, applying the organization's brand,
  security, compliance and UX skills and flagging areas of concern. Use when
  someone says "spec this intent", "write the spec for this intent", "create
  spec.md", or asks for a requirements and design spec from an intent. Not
  for PRDs, user stories or implementation plans (plan.md).
---
# Write a spec.md

A spec.md turns an accepted intent into requirements and a design the
engineering team can plan against. Requirements and design are written
together in one session, with the organization's policies applied while
writing and every conflict flagged. The product owner reviews the spec but
does not write it. It is not an implementation plan: no file-by-file task
list, no estimates. That comes later, in plan.md.

## 1. Read the intent
Read the intent.md you were given. If none was given, list `intent/` and
ask which one. Only spec an accepted intent, meaning it is merged into the
main branch. If it is still in an open pull request or you cannot find it,
stop and say so. Do not invent an intent.

## 2. Read the template and example
Read `template.md` in this skill's folder. Read `example.md` too if you
have not written a spec in this session yet. It shows the level of detail
to aim for. Never copy its content.

## 3. Load the organization's policy skills
Find the skills available for brand, security, compliance and UX, and
read each one that applies to this intent before writing. For each skill
you apply, record its version: a version in its frontmatter if it has one,
otherwise the short SHA of the last commit to its folder, otherwise
"unknown". If no skill exists for one of those four areas, say so and add
it as an area of concern.

## 4. Fit it to the existing codebase
If you have access to the code, explore the parts the intent touches so
the design fits what exists. Read only. Do not change code. Use the
systems named in the intent or found in the code. Do not invent systems,
endpoints or policies. If the product owner made a mockup (for example in
Claude Design), link it in the Design section and design to match it.

## 5. Write the spec
Follow `template.md` exactly: same headings, same order. Replace every
placeholder in angle brackets. Then check:
- Every requirement is testable.
- Every constraint in the intent has a row in the constraints table.
- Every open question in the intent is answered or carried forward as a
  numbered concern.
- Where two policies contradict, or a constraint cannot be met, you have
  not picked a silent winner. Raise a concern that names both sources,
  proposes options, and names the policy owner.
- Risk is "higher" if the change touches anything the organization
  classes as higher risk (for example payments, personal data,
  authentication or regulated reporting), or if you are unsure.

Write "None." under Areas of concern only when every constraint is met and
no policies conflict.

## 6. Review with the product owner
If someone is present, walk them through the areas of concern first, then
ask: "Does this solve the problem in the intent?" Record their decisions
in the spec. A concern stays open until the product owner resolves it with
its owner. Do not mark a concern resolved on their behalf.

If nobody is present (for example in a CI job triggered by the intent
merge), ask no questions. Put everything you would have asked into Areas
of concern and continue.

## 7. Commit next to the intent
- File name: the intent's file name with `.spec.md` instead of `.md`. For
  `intent/2026-10-claims-status.md`, write
  `intent/2026-10-claims-status.spec.md`.
- If git access or a GitHub connector is available, create a branch named
  `spec/<kebab-case-name>`, add only the spec, and open a pull request
  titled "Spec: <short name>". In the description, list the open concerns
  and their owners, and say whether a technical lead must review.
- If no repo access is available, give the product owner the final file
  and tell them where it goes.

## Rules
- Leave Status as draft. The product owner's decision is recorded by the
  pull request: merging accepts the spec for build, closing rejects it.
  For higher-risk specs, the product owner consults a technical lead
  first. A human always makes this call.
- Never edit the intent. If it is wrong or incomplete, raise a concern.
- Do not start implementation or write plan.md.
- Never include secrets, passwords or customer personal data. Describe the
  kind of data instead ("customer addresses").
