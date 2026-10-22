# AI Native SDLC

## What is the SDLC?
The process that takes software from an idea to a product.

The process has **six** stages: Planning, Designing, Building, Testing, Deploying and Maintaining.

## OLD SDLC
+ Plan: Requirements gathering.
+ Design: Specs (detailed written requirements and design documents).
+ Build: Test and Code (handwritten), and documentation written after the development happens.
+ Test: QA gates at stage boundaries.
+ Deploy: Humans review every line of code and governance occurs in review cycles and often inconsistently.
+ Maintain: Humans watch production for bugs.

## **HUMANS ARE ACCOUNTABLE FOR EVERY DECISION THAT REQUIRES JUDGEMENT**

## AI Native SDLC

### PLAN PHASE
**Description**: The user, working with AI, synthesizes pain points straight from the sources and captures them in an `intent.md`. This file is readable by humans and actionable by machines.

- **Deliverable**: `intent.md`
- **Who**: Product owner watches and approves, tech team stands up the intent in the repo.
- **Where**: In the product repo. If the intent covers many repos, you may want to create an intent repo to store it.
- **Prerequisites**: An idea and a brainstorm
- **Automation**: Use the `write-intent` skill in this repo, or Matt Pocock’s `grill-me` skill, to run the brainstorm and write the `intent.md`.

You start with an idea and brainstorm with your preferred agent. The agent asks the questions an analyst would ask, such as scope, users, constraints and what success looks like. The intent covers the problem, proposed outcome, affected users and systems, constraints and open questions.

Or you can start with **Matt Pocock’s** [`grill-me`](https://github.com/mattpocock/skills/blob/main/skills/productivity/grilling/SKILL.md) skill. It is an interview skill that stress-tests plans and designs through systematic questioning, going through a decision tree branch by branch until a shared understanding is reached.

Once the brainstorming or `grill-me` session is done, we create an `intent.md` file. This file does not have to be in the repo. It can be in a ticket (Jira).

The product owner approves, and the accept or reject decision that sends the intent into Stage 2: Design is recorded as the merge or the closing review.

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

```markdown
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

### DESIGN PHASE
**Description:** The user, working with an agent, merges requirements and design into one working session, guided by standards encoded as skills and stored in a Git repo.

- **Deliverable**: `spec.md`
- **Who**: Product owner, using the `intent.md`
- **Where**: In the intent/ folder, next to the `intent.md`
- **Prerequisites**: `intent.md`
- **Automation**: The `spec.md` creation can be automated: merging the accepted `intent.md` runs a script or GitHub workflow that opens the `spec.md` pull request.

You can use the following prompt to create the `spec.md` file:

```
Read the attached intent.md and produce a requirements and design spec for integrating it into our existing codebase. Apply the skills available to you so the plan conforms to our brand guidelines, security policies and UX standards. Document the spec fully as spec.md, ready to hand to the engineering team. Describe clearly any areas of concern, especially where you cannot satisfy contradicting policies.
```

You can also use **Matt Pocock’s** [`to-spec`](https://github.com/mattpocock/skills/blob/main/skills/engineering/to-spec/SKILL.md) skill or the skills in this repo.

### BUILD PHASE
**Description:** Code and tests are generated using AI. Institutional knowledge, as `AGENT.md` (`CLAUDE.md` -> `AGENT.md`) and skills, is stored in the Git repo.

- **Deliverable**: `plan.md`
- **Who**: Engineers
- **Where**: Codebase repo
- **Prerequisites**: `intent.md`, `spec.md` and `AGENT.md` (`CLAUDE.md` -> `AGENT.md`)
- **Automation**: When **implementation** diverts from the **plan**, update `plan.md` in the same commit. Use a hook for synchronization between the two (implementation and plan).

The engineer opens their agent in plan mode and uses the approved `spec.md` and `intent.md` files. Iterate until the engineer is happy with the plan. Commit the plan once ready as `plan.md`. If implementation diverges from the plan, make sure the `plan.md` file is updated.  You can use hooks to sync the implementation with the plan.

The engineer interrogates the plan by asking what could break with the change, which step is most risky, and what other options there are.

```markdown
# Plan: claims status self-service (from intent.md 2026-06-02)

## Files that change
portal/src/claims/StatusPanel.tsx (new), claims-api/routes/status.py,
claims-api/tests/test_status.py

## Order of work
1. Add the status endpoint behind existing auth (R1, R2, R3).
2. Panel against the endpoint (R1, R4).
3. Wire into the portal nav.

## Risks
The claims-core API rate-limits at 50 rps; the panel must cache.

## Proof
test_status.py covers the four claim states (R2); screenshot matches the
approved mock.
```

**R1, R2, R3, R4** are requirements from the `spec.md`.

#### `AGENT.md` (`CLAUDE.md` -> `AGENT.md`)

The `AGENT.md` (`CLAUDE.md` -> `AGENT.md`) file covers conventions, commands, architecture, and the mistakes the team sees most often. Knowledge that used to sit in people's heads and on wikis becomes a file the agent reads at the start of every session, maintained by the whole team and iterated on whenever a mistake is made.


```markdown
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

#### Skills
A skill is a packaged set of instructions that teaches AI how your organization does a specific kind of work. It captures reusable company, technical and institutional knowledge, such as brand guidelines, security policies or UX standards. Skills are version controlled and shared across every team. When a policy changes, the skill is updated once, and every team's work follows the new standard consistently.


#### Hooks
Hooks are automatic rules that run at specific moments while Claude Code works. You can use hooks to run checks such as linting or removing sensitive data.

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          { "type": "command",
            "command": "${CLAUDE_PROJECT_DIR}/.claude/hooks/api-policy.sh" }
        ]
      }
    ]
  }
}
```

```bash
#!/bin/bash
# Backs rule 4 of the secure-api-review skill with the same script the skill runs.
file=$(jq -r '.tool_input.file_path')
case "$file" in
  "$CLAUDE_PROJECT_DIR"/claims-api/routes/*.py) ;;
  *) exit 0 ;;                                   # not an endpoint
esac
cd "$CLAUDE_PROJECT_DIR" || exit 2
if ! found=$(scripts/check-endpoints.sh "${file#"$CLAUDE_PROJECT_DIR"/}"); then
  echo "secure-api-review rule 4: a PII field reaches a log or an error message." >&2
  echo "$found" >&2
  exit 2
fi
exit 0
```

#### Parallel session and subagents
- A parallel session is a full agent instance, and each one works on its own task in its own Git worktree.
- Subagents are helpers that an agent sends off to handle parts of a job. They are started from a single session, and each one works in its own separate context.

- An engineer can run multiple agent sessions at once, each with its own worktree.
- An engineer can create their own subagents to run repeated jobs.
- The engineer becomes an orchestrator.

**WATCH OUT: HIGH TOKEN USE**

Each subagent starts fresh. It doesn’t know what the main agent already knows. So each one has to:

- read its own instructions
- read the files it needs, even if the main agent already read them
- write its own report back

### TEST PHASE
**Description**: Sessions check their own work before a human sees it, and the configuration that steers the agent gets regression-tested like the code it writes.

- **Deliverables**: Tests
- **Who**: Engineers
- **Where**: Repo
- **Prerequisites**: `AGENT.md` with the build, test and lint commands

#### In AGENT.md (CLAUDE.md -> AGENT.md)

```markdown
## Verifying your work

- Build: make build (must finish with "Build succeeded")
- Test: make test (all green; never skip or delete a failing test)
- Lint: make lint (zero warnings)

Run all three before reporting any task complete, and paste the output.
If a test fails, fix the code, not the test.
```

Make the agent verify its own work by adding commands to your `AGENT.md` (`CLAUDE.md` -> `AGENT.md`) and giving it clear, measurable goals. You can also use the TDD approach (red, green, refactor).

### DEPLOYMENT PHASE
**Description**: Review runs in both directions, and governance is enforced as the agent acts. The agent does everything up to the production gate and nothing past it.

- **Deliverables**: `REVIEW.md`
- **Who**: Tech lead 
- **Automation**: Code reviews run when someone opens a PR, using `REVIEW.md`

#### REVIEW.md Example
```markdown
# Review instructions
## Passes
Run three passes and tag each finding with its pass:
- Bugs: logic errors, broken edge cases, subtle regressions
- Security: injection risks, authentication gaps, PII in logs
- Compliance: the change matches spec.md, plan.md and our design principles
## What Important means here
Reserve Important for findings that would break behavior, leak data
or breach a policy. Style and naming are nits.
## Cap the nits
Report at most five nits per review; summarize the rest as a count.
## Do not report
Generated files under src/gen/ and anything CI already enforces.
```

Use already established bots, skills, agents and hooks for reviews.

### MAINTENANCE
**Description**: The loop closes. A trigger invokes Claude with no person in the invocation path, and what it finds re-enters the pipeline as intent.md.

A trigger such as a control-band breach, a ticket, a channel message or a schedule invokes Claude without a person in the path.
