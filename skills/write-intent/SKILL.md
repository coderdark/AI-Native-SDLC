---
name: write-intent
description: Turn a rough idea into an intent.md by brainstorming with the user like an analyst until the problem, outcome, users, constraints and open questions are concrete.
---

# Write intent

Help the user turn a rough idea into a short, clear `intent.md`. The user can talk informally. Your job is to ask the questions an analyst would ask, until the idea is concrete enough to hand to Stage 2: Design.

## When to use

- The user describes a problem, a pain point, or an idea and wants to capture it as an intent.
- The user says things like "we can't do X today", "people keep asking for Y", or "it would be better if Z".
- The user asks for an `intent.md`.

## What the user may give you

Any mix of these, in any order, in plain words:
- what they cannot do today
- who is affected
- what better looks like
- what is out of scope

No formal language is needed. Do not ask the user to rewrite anything formally.

## Steps

1. **Listen first.** Read what the user gave you. Sort it into the five sections below. Notice what is missing or vague.

2. **Brainstorm until it is concrete.** Ask the questions an analyst would ask. Ask **two or three at a time**, not a long list, and build on each answer. Cover:
   - **Problem:** What exactly goes wrong today? How often? What does it cost (time, money, complaints)? Is there a number, even a rough one?
   - **Outcome:** What does better look like? How would someone know it worked? What is the smallest version that would still help?
   - **Users:** Who feels the problem? Who would use the change? Who else is touched (support, other teams, partners)?
   - **Systems:** Which apps, services or data are involved?
   - **Constraints:** Rules that must not be broken: security, privacy, legal, budget, deadlines, technology.
   - **Scope:** What is deliberately left out?

   Stop asking when each section has something specific in it. Do not keep asking just to be thorough.

3. **Keep the user's facts, not your guesses.** Write down only what the user told you or confirmed. If something is still unknown, put it under Open questions. Never invent numbers, names or systems.

4. **Write the intent.** Use the format below. Keep it short: a few plain sentences per section. Use the user's own words where they are clear.

5. **Show it and confirm.** Show the draft to the user and ask if anything is wrong or missing. Update it, then save it as `intent.md`.

## Output format

```
# Intent: <short name>
Author: <name> (<team>). Status: draft.
## Problem
<What goes wrong today, for whom, and how much it costs.>
## Proposed outcome
<What better looks like.>
## Affected users and systems
<People, teams and systems involved.>
## Constraints
<Rules that must not be broken. What is out of scope.>
## Open questions
<Anything still unknown.>
```

## Rules

- Describe the problem and the outcome, not the solution. Leave the "how" for the design stage.
- If the user starts designing (for example, "add a button that..."), note the underlying need and keep the design idea as context, not a requirement.
- Status is always "draft". The product owner decides if it moves on.
- If the author or team is unknown, ask once. If still unknown, leave a placeholder.

## Example

```
# Intent: claims status self-service
Author: J. Doe (claims operations). Status: draft.
## Problem
Customers phone the contact center to ask where their claim is.
Handlers spend roughly a third of call time on status-only queries.
## Proposed outcome
Customers see claim status, next step and expected date in the portal.
## Affected users and systems
Claims handlers, portal team, claims-core API.
## Constraints
No new PII in the portal session. Existing authentication only.
## Open questions
Do third-party loss adjusters need access too?
```