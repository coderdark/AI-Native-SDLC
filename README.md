# AI-Native SDLC

## What is the SDLC?
The SDLC (software development life cycle) is the process that takes software from an idea to a product.

It has **six** phases: Plan, Design, Build, Test, Deployment and Maintenance.

## The old way

| Phase | Who | How it works |
| :---- | :---- | :---- |
| Plan | Product managers and business analysts, with stakeholders | Requirements gathering. |
| Design | Business analysts, architects and designers | Specs (detailed written requirements and design documents). |
| Build | Developers | Code and tests are written by hand. Documentation is written after development. |
| Test | QA testers | Work is checked only between phases (QA gates). |
| Deployment | Senior developers (code review) and the operations or release team | Humans review every line of code. Rules are checked in review meetings, often inconsistently. |
| Maintenance | Operations and support teams | Humans watch production for bugs. |

> **Humans are accountable for every decision that requires judgement.**

## The AI-native way

| Phase | Deliverable |
| :---- | :---- |
| Plan | `intent.md` |
| Design | `spec.md` |
| Build | `plan.md` |
| Test | Tests |
| Deployment | `REVIEW.md` |
| Maintenance | A new `intent.md` |

### Plan Phase
**Description**: You work with an agent to collect pain points straight from the sources and write them down in `intent.md`. People can read this file, and agents can act on it.

- **Deliverable**: `intent.md`
- **Who**: The product owner works with the agent and approves the result. The tech team adds it to the repo.
- **Where**: In the product repo, in a separate intent repo (if it covers many repos), or in a ticket (for example, Jira).
- **Prerequisites**: An idea
- **Automation**: The `write-intent` skill in this repo can run the brainstorm and write `intent.md` for you.

**How it works:**

1. Start with an idea and brainstorm with your agent. The agent asks the questions an analyst would ask: scope, users, constraints and what success looks like.
2. Or start with **Matt Pocock’s** [`grill-me`](https://github.com/mattpocock/skills/blob/main/skills/productivity/grilling/SKILL.md) skill. It interviews you, branch by branch, until the plan is clear.
3. When the brainstorm is done, create the `intent.md` file. It covers the problem, proposed outcome, affected users and systems, constraints and open questions.
4. The product owner approves or rejects it. Approving means merging it; rejecting means closing the review. Only an approved intent moves on to the Design phase.

**Prompt to create the `intent.md` file:**

```
That is clear enough. Now write the file at <YOUR_PATH>/intent.md.
Line 1 is "# Intent: name of the change".
Line 2 is "Author: my name (my team). Status: draft."
Then five level-two headings, in this order, which cannot change:
Problem, Proposed outcome, Affected users and systems,
Constraints, Open questions.
Write only what I said. Put anything I did not say into Open questions. Do not invent it.
```

**Example `intent.md`:**

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

---

### Design Phase
**Description**: You work with an agent to write the requirements and the design together, in one session. Skills make sure it follows your company’s standards.

- **Deliverable**: `spec.md`
- **Who**: The product owner, starting from the approved `intent.md`
- **Where**: In the same folder as `intent.md`
- **Prerequisites**: `intent.md`
- **Automation**: This can run on its own. When an `intent.md` is approved (merged), a script or GitHub workflow creates `spec.md` as a pull request.

**Prompt to create the `spec.md` file:**

```
Read the attached intent.md and produce a requirements and design spec for integrating it into our existing codebase. Apply the skills available to you so the plan conforms to our brand guidelines, security policies and UX standards. Document the spec fully as spec.md, ready to hand to the engineering team. Describe clearly any areas of concern, especially where you cannot satisfy contradicting policies.
```

You can also use **Matt Pocock’s** [`to-spec`](https://github.com/mattpocock/skills/blob/main/skills/engineering/to-spec/SKILL.md) skill or the skills in this repo.

---

### Build Phase
**Description**: The agent writes the code and tests. The team’s knowledge lives in `AGENT.md` and skills, stored in the Git repo.

- **Deliverable**: `plan.md`
- **Who**: Engineers
- **Where**: The code repo
- **Prerequisites**: `intent.md`, `spec.md` and `AGENT.md`
- **Automation**: A hook can check that `plan.md` and the code stay in sync.

**How it works:**

1. The engineer opens the agent in plan mode with the approved `spec.md` and `intent.md`.
2. The engineer questions the plan: What could break? Which step is riskiest? What other options are there?
3. When the plan is ready, commit it as `plan.md`.
4. If the code later changes from the plan, update `plan.md` in the same commit.

**Example `plan.md`:**

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

#### `AGENT.md`

`AGENT.md` is the file the agent reads at the start of every session. It holds the team’s commands, conventions, architecture, and the mistakes the agent makes most often. Knowledge that used to live in people’s heads and on wikis becomes a file. The whole team keeps it up to date and adds to it whenever a mistake happens.

> Claude Code looks for `CLAUDE.md`, so link `CLAUDE.md` to `AGENT.md` (`CLAUDE.md` -> `AGENT.md`).

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
A skill is a packaged set of instructions that teaches the agent how your company does a certain kind of work, for example brand guidelines, security policies or UX standards. Skills are stored in Git and shared by every team. When a policy changes, you update the skill once, and every team follows the new rule.

#### Hooks
Hooks are automatic checks that run at set moments while the agent works, for example after it edits a file. They run every time, so the agent can’t skip them. Use them for checks like linting or keeping sensitive data out of logs.

In this example, after every file edit, a script checks that no private data (PII) ends up in a log or an error message. If it does, the hook tells the agent to fix it.

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

#### Parallel sessions and subagents
- **Parallel sessions**: several full agents running at once. Each one works on its own task, in its own copy of the code (a Git worktree).
- **Subagents**: helpers that one agent sends off to do part of a job. Each one works separately and reports back.
- Engineers can create their own subagents for jobs they repeat.
- The engineer’s role becomes directing the agents, like an orchestra conductor.

> **Watch out: high token use.**

Each subagent starts fresh. It doesn’t know what the main agent already knows. So each one has to:

- read its own instructions
- read the files it needs, even if the main agent already read them
- write its own report back

---

### Test Phase
**Description**: The agent checks its own work before a person sees it.

- **Deliverable**: Tests
- **Who**: Engineers
- **Where**: The code repo
- **Prerequisites**: `AGENT.md` with the build, test and lint commands

#### In `AGENT.md`

Add your build, test and lint commands to `AGENT.md`, with clear goals the agent can check. Then the agent can test its own work. You can also use TDD (red, green, refactor): write a failing test first, then make it pass.

```markdown
## Verifying your work

- Build: make build (must finish with "Build succeeded")
- Test: make test (all green; never skip or delete a failing test)
- Lint: make lint (zero warnings)

Run all three before reporting any task complete, and paste the output.
If a test fails, fix the code, not the test.
```

---

### Deployment Phase
**Description**: The agent reviews other people’s code and fixes the comments on its own code. Rules are checked automatically while the agent works. The agent does everything up to the release to production. A person approves the release.

- **Deliverable**: `REVIEW.md`
- **Who**: Tech lead
- **Automation**: When someone opens a PR, the code review runs automatically, using `REVIEW.md`.

#### `REVIEW.md` example

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

Reuse the review bots, skills, agents and hooks you already have.

---

### Maintenance Phase
**Description**: The loop closes. Problems found in production become new work, without anyone having to start it.

- **Deliverable**: A new `intent.md`
- **Who**: People sort and review what the agent finds. They no longer have to start the work.
- **Automation**: A trigger starts an agent. The agent investigates the problem and writes what it finds as a new `intent.md`.

A trigger can be:

- a number (like the error rate) moving outside its normal range
- a ticket
- a message in a team channel
- a schedule

The agent can only act through approved, safe routes, such as opening a pull request for review. It can’t change production on its own.

The loop starts again: the new `intent.md` goes through all the phases again.

---

## Sources

- **Anthropic, [The AI-Native SDLC Playbook](https://academy.claude.com/courses/ai-native-sdlc-playbook)** (Claude Academy). The phases, the ideas in each phase, and the examples (`intent.md`, `plan.md`, `AGENT.md`, `REVIEW.md`, the prompts and the hook) are based on this course.
- **Matt Pocock, [skills](https://github.com/mattpocock/skills)** (GitHub). The `grill-me` and `to-spec` skills.
- **Anthropic, [Claude Code documentation](https://code.claude.com/docs)**. Hooks, subagents and `CLAUDE.md`.
