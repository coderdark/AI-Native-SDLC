# AI Native SDLC

## What is SDLC: 
Process that takes software from idea to products

The process has **6** (six) stages, Planning, Design, Building, Testing, Deploying and Maintaining.

## OLD SDLC
+ Plan: Requirements gathering.
+ Design: Specs (detailed written requirements and design documents).
+ Build: Test and Code (handwritten), and documentation written after the development happens.
+ Test: QA gates at stage boundaries.
+ Deploy: Humans review every line of code and governance occurs in review cycles and often inconsistently.
+ Maintain: Humans watch production for bugs.

## AI Native SDLC

### PLAN PHASE
**Description**: User using AI synthesizes pain points straight from the sources and captures them in an intent.md.  This file is human and machine actionable

- **Deliverable**: `intent.md`
- **Who**: Product owner watches, tech team stands up the intent in the repo.
- **Where**: in product repo or if the intent covers many repos, then we may one to create an intent repo to store the intent.

You start with an idea, brain storm with your prefer agent: 

Or you can start with the __Matt Pocock’s__ `grill-me` skill, https://github.com/mattpocock/skills/blob/main/skills/productivity/grilling/SKILL.md this is an interview skill that stress-tests plans and designs through systematic questioning.  Goes through a decision tree branch by branch until a shared understanding is reached.

Once done with brainstorming or using the `grill-me` skill session, we create an `intent.md` file.  This file does not have to be in the repo. It can be in a ticket (Jira).

Prompt to create the `intent.md` file:

```
That is clear enough. Now write the file at <YOUR_PATH>/intent.md.
Line 1 is "# Intent: name of the change".
Line 2 is "Author: my name (my team). Status: draft."
Then five level-two headings, in this order, which cannot change:
Problem, Proposed outcome, Affected users and systems,
Constraints, Open questions.
Write only what I said. Put anything I did not say into Open questions. Do not invent it.
```

Example `intent.md`

```# Intent: claims status self-service
Author: J. Ortiz (claims operations). Status: draft.

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

### DESIGN PHASE
Description: User using an agent merges requirements and design into a working session guided by standards encoded as skills, stored in git repo.

Deliverable: `spec.md`

You can use the following prompt to create the `spec.md` file:

```
Read the attached intent.md and produce a requirements and design spec for integrating it into our existing codebase. Apply the skills available to you so the plan conforms to our brand guidelines, security policies and UX standards. Document the spec fully as spec.md, ready to hand to the engineering team. Describe clearly any areas of concern, especially where you cannot satisfy contradicting policies.
```

You can also use __Matt Pocock’s__ `to-spec` skill.

https://github.com/mattpocock/skills/blob/main/skills/engineering/to-spec/SKILL.md

### BUILD PHASE
Description: Code and test are generated using AI and institutional knowledge as AGENT.md (CLAUDE.md -> AGENT.md) and skills are stored in git repo.

Deliverable: `plan.md`

The user opens their agent in plan mode and uses the `spec.md` and `intent.md` files.  Iterate until the engineer is happy with the plan. Commit the plan once ready as `plan.md`. If implementation diverges from the plan, make sure the `plan.md` file is updated.  You can use hooks to sync the implementation with the plan.

The AGENT.md file will cover: conventions, commands, architecture, and the mistakes the team sees most often. Knowledge that used to sit in people's heads and on wikis becomes a file the agent reads at the start of every session, maintained by the whole team and iterated on whenever a mistake is made.

```
# Plan: claims status self-service (from intent.md 2026-06-02)

## Files that change
portal/src/claims/StatusPanel.tsx (new), claims-api/routes/status.py,
claims-api/tests/test_status.py

## Order of work
1. Add the status endpoint behind existing auth.
2. Panel against the endpoint.
3. Wire into the portal nav.

## Risks
The claims-core API rate-limits at 50 rps; the panel must cache.

## Proof
test_status.py covers the four claim states; screenshot matches the
approved mock.
```

AGENT.md (CLAUDE.md -> AGENT.md)

```
# Payments service

## Commands
- Build: make build
- Test: make test (unit), make itest (integration, needs docker)
- Lint: make lint (runs in CI; fix before pushing)

## Conventions
- Java 21, Spring Boot 3. No new Lombok.
- Money is always BigDecimal, never double.
- Every endpoint needs an integration test in src/itest.

## Architecture
- api/ holds REST controllers, core/ holds domain logic,
  adapters/ talks to external systems.
- Kafka events are defined in schemas/; never edit generated classes.

## Things Claude gets wrong
- Do not bump dependency versions; the platform team owns them.
- The legacy v1/ package is frozen; changes go in v2/.
```

### TEST PHASE
Description: Sessions checks their own work before a human sees it, and the configuration that steers the agent gets regression-tested like the code it writes.

Deliverables: AGENT.md session for test

```## Verifying your work

- Build: make build (must finish with "Build succeeded")
- Test: make test (all green; never skip or delete a failing test)
- Lint: make lint (zero warnings)

Run all three before reporting any task complete, and paste the output.
If a test fails, fix the code, not the test.
```

### DEPLOYMENT PHASE
Description: Review runs in both directions, and governance is enforced as the agent acts. The agent does everything up to the production gate and nothing past it.
