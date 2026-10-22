---
name: intent-to-spec
description: Turn an intent.md into a requirements and design spec (spec.md) for the existing codebase, applying brand, security and UX skills and flagging policy conflicts.
---

# Intent to spec

Read an `intent.md` and produce `spec.md`: a requirements and design spec for building the intent into the existing codebase. The spec must follow the organization's brand guidelines, security policies and UX standards, and be ready to hand to the engineering team.

## When to use

- The user attaches or points to an `intent.md` and asks for a spec, a design, or a plan for engineering.
- The user asks to move an intent from Stage 1: Plan into Stage 2: Design.

If no `intent.md` is attached or reachable, ask for it. Do not write a spec from memory or guesswork.

## Steps

1. **Read the intent fully.** Note the author, the problem, every requirement, every constraint and every open question. Keep the author's wording for constraints so they can be traced later.

2. **Find and load the policy skills.** List the skills available in this session. Load every skill that covers brand or voice, security, or UX, and any other skill whose description touches the intent's subject. Record the names of the skills you applied.
   - If no skill exists for brand, security or UX, do not invent a policy. Record the gap as an area of concern.

3. **Study the existing codebase.** Find the services, endpoints, components, schemas and auth mechanisms the change will touch. Prefer extending what exists over adding new parts. Every design claim must name a real file, service or component. If something cannot be confirmed in the code, say so.

4. **Write the requirements.** Number them R1, R2 and so on. Each one must be:
   - one sentence,
   - testable (someone can check pass or fail),
   - free of design detail (say what, not how).

5. **Write the design.** Describe how the requirements are met in this codebase: new or changed endpoints, data flow, components, caching, and auth. Keep it short and concrete. Refer to requirements by number.

6. **Trace every constraint.** Build a table with one row per constraint from the intent and from the applied skills. Each row says exactly which requirement or design choice satisfies it. A constraint with no row is a gap.

7. **Carry forward open questions.** List each open question from the intent. Say whether it is answered by the spec, and if not, link it to an area of concern.

8. **Write the areas of concern.** Number them. Include:
   - **Conflicts between policies.** When two skills or constraints contradict each other, do not silently pick one. State both rules, say plainly that both cannot be satisfied as written, propose a resolution, and name the owner who must decide.
   - Anything left out of scope to protect a constraint, and why.
   - Risks not yet verified (load, performance, data quality, dependencies).
   - Missing skills or policies.
   Every concern names an **owner** (a team or role).

9. **Check before finishing.** Use the checklist below. Then save the result as `spec.md`, next to `intent.md` if there is one.

## Output format

```
# Spec: <short name>
From: intent.md (<author>, <team>). Status: ready for product owner review.
Skills applied: <skill names>.

## Requirements
- R1. ...

## Design
- ...

## How each constraint in the intent is met
| Constraint | Met by |
| :---- | :---- |
| ... | ... |

## Open questions from the intent
- <question>. <Answered, or: Not answered. Carried forward as concern N.>

## Areas of concern
1. <concern>. Owner: <team or role>.
```

## Checklist

- Every requirement is numbered, one sentence and testable.
- Every design point names a real part of the codebase.
- Every constraint from the intent and the applied skills has a row in the table.
- Every open question appears, answered or carried forward.
- Every policy conflict is stated openly, with both rules, a proposed resolution and an owner.
- No policy has been invented to fill a gap.
- Status line says "ready for product owner review", not "approved".

## Example

```
# Spec: claims status self-service
From: intent.md (J. Doe, claims operations). Status: ready for product owner review.
Skills applied: secure-api-review, brand-voice, portal-ux.

## Requirements
- R1. A signed-in customer sees status, next step and expected date for each open claim.
- R2. Status is one of four states: received, in review, approved, paid.
- R3. The response carries those three fields and nothing else.
- R4. What the customer sees is at most 60 seconds old.

## Design
- New endpoint GET /claims/{id}/status in claims-api, behind the gateway JWT.
- claims-api reads the claim from claims-core. The portal never calls claims-core.
- New StatusPanel on the portal claims page. It caches each answer for 60 seconds.

## How each constraint in the intent is met
| Constraint | Met by |
| :---- | :---- |
| No new PII in the portal session | R3. None of the three fields is tagged pii in the schema. |
| Existing authentication only | The gateway JWT. No new login, token or role. |

## Open questions from the intent
- Do third-party loss adjusters need access too? Not answered. Carried forward as concern 1.

## Areas of concern
1. Loss adjusters have no portal account. Giving them access breaks "existing
   authentication only", so they are left out of this spec. Owner: security.
2. The portal-ux skill says every status shows a date. The brand-voice skill says
   never to state a payment date as a commitment. I cannot satisfy both as written.
   Proposed label: "Estimated date". Owner: compliance.
3. claims-core allows 50 requests a second. The 60-second cache should keep the
   portal under that, but nobody has load tested it. Owner: claims-core team.
```