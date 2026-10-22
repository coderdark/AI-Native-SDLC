# The AI-Native SDLC

## What is the SDLC?

The software development life cycle (SDLC) is the process that takes software from an idea to a working product.

It has six stages: **Plan, Design, Build, Test, Deploy and Maintain.**

## The traditional SDLC

| Stage | How it works |
| :---- | :---- |
| Plan | Requirements are gathered by committee through workshops and sign-offs, and written up by hand. |
| Design | Analysts write detailed specs, and designers interpret them. |
| Build | Code and tests are handwritten. Documentation is written after development. |
| Test | QA gates at stage boundaries: work is checked only when it moves from one stage to the next. |
| Deploy | Humans review every line of code. Governance happens in review cycles, often inconsistently. |
| Maintain | Humans watch production for bugs. |

## The AI-native SDLC at a glance

| Stage | Deliverable | Who | Key idea |
| :---- | :---- | :---- | :---- |
| Plan | `intent.md` | Product owner | Capture the problem in a file people and agents can both read. |
| Design | `spec.md` | Product owner | Requirements and design in one session, guided by skills. |
| Build | `plan.md`, code and tests | Engineers | Plan first, then build with `AGENTS.md`, skills, hooks and subagents. |
| Test | Passing tests and an eval suite | Engineers, platform engineer | The agent checks its own work, and its setup is regression-tested. |
| Deploy | `REVIEW.md`, approval hooks | Tech lead | AI reviews every PR. Hooks enforce governance. Humans approve. |
| Maintain | A new `intent.md` | Service owner | Problems in production re-enter the loop automatically. |

> **Humans are accountable for every decision that requires judgement.**

---

## 1. Plan

**What changes:** Instead of weeks of workshops, the product owner works with an agent that pulls pain points straight from the sources and captures them in `intent.md`. The file is readable by people and actionable by agents.

- **Deliverable:** `intent.md`
- **Who:** The product owner writes and approves it. The tech team stands it up in the repo.
- **Where:** In the product repo. If the intent covers several repos, keep it in a dedicated intent repo. It can also live in a ticket (for example, Jira).
- **Prerequisites:** An idea.
- **Automation:** Use the `write-intent` skill in this repo to run the brainstorm and write the file.

### How to do it

1. Start with the idea and brainstorm with your agent. The agent asks the questions an analyst would ask: scope, users, constraints and what success looks like.
2. Alternatively, use Matt Pocock's [`grill-me`](https://github.com/mattpocock/skills/blob/main/skills/productivity/grilling/SKILL.md) skill. It stress-tests a plan through systematic questioning, branch by branch, until there is a shared understanding.
3. When the idea is concrete, ask the agent to write `intent.md`. It covers the problem, proposed outcome, affected users and systems, constraints and open questions.

**Prompt to create `intent.md`:**

```text
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

## 2. Design

**What changes:** Requirements and design are compressed into one working session with an agent, guided by standards encoded as skills and versioned in Git.

- **Deliverable:** `spec.md`
- **Who:** The product owner, starting from the accepted `intent.md`
- **Where:** In the same folder as `intent.md`
- **Prerequisites:** An accepted `intent.md`
- **Automation:** Use the `intent-to-spec` skill in this repo. To go further, make merging an accepted `intent.md` the trigger: a GitHub workflow runs the spec pass without a person and opens `spec.md` as a pull request. The product owner's first job becomes the review.

**Prompt to create `spec.md`:**

```text
Read the attached intent.md and produce a requirements and design spec for integrating it into our existing codebase. Apply the skills available to you so the plan conforms to our brand guidelines, security policies and UX standards. Document the spec fully as spec.md, ready to hand to the engineering team. Describe clearly any areas of concern, especially where you cannot satisfy contradicting policies.
```

You can also use Matt Pocock's [`to-spec`](https://github.com/mattpocock/skills/blob/main/skills/engineering/to-spec/SKILL.md) skill or the skills in this repo.

**The path to automation:**
1. Run the prompt by hand until the results are reliable.
2. Save it as an organization-level slash command, for example `/spec`.
3. Trigger it automatically when an `intent.md` is merged.

---

## 3. Build

**What changes:** Code and tests are generated by AI. Institutional knowledge lives in versioned, machine-readable files (`AGENTS.md`) and skills, instead of in people's heads and on wikis.

- **Deliverables:** `plan.md`, code and tests
- **Who:** Engineers
- **Where:** The codebase repo
- **Prerequisites:** `intent.md`, `spec.md` and `AGENTS.md`
- **Automation:** When the implementation diverges from the plan, update `plan.md` in the same commit. A hook can check that the two stay in sync.

### Start in plan mode

1. Open the agent in plan mode with the approved `spec.md` and `intent.md`.
2. Interrogate the plan: what could break, which step is riskiest, and what the alternatives are.
3. Iterate until you are happy, then commit it as `plan.md`.

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

R1 to R4 are the numbered requirements in `spec.md`.

### `AGENTS.md`

`AGENTS.md` holds the team's conventions, commands, architecture and the mistakes the agent makes most often. The agent reads it at the start of every session. The whole team maintains it and updates it whenever a mistake repeats.

> **Note:** Claude Code reads `CLAUDE.md`. If your repo uses `AGENTS.md` so that several tools can share it, add a `CLAUDE.md` containing the line `@AGENTS.md` to import it.

**Example `AGENTS.md`:**

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

## Things the agent gets wrong
- Do not bump dependency versions; the platform team owns them.
- The legacy v1/ package is frozen; changes go in v2/.
```

### Skills

A skill is a packaged set of instructions that teaches the agent how your organization does a specific kind of work. It captures reusable company, technical and institutional knowledge, such as brand guidelines, security policies or UX standards. Skills are version controlled and shared across every team. When a policy changes, the skill is updated once, and every team's work follows the new standard consistently.

This repo includes two skills:
- [`skills/write-intent`](skills/write-intent/SKILL.md): turns a rough idea into `intent.md`
- [`skills/intent-to-spect`](skills/intent-to-spect/SKILL.md): turns `intent.md` into `spec.md`

### Hooks

Hooks are automatic checks that run at set moments while the agent works, for example before or after it edits a file. Unlike instructions, they run every time and the agent cannot skip them. A hook can **allow** an action, **block** it, or **ask** a person to approve it. Use them for linting, formatting, protecting files and keeping sensitive data out of logs.

- `exit 0` allows the action.
- `exit 2` blocks it (or, after an edit, sends the problem back to the agent to fix), and the message is shown to the agent.

**Example: `.claude/settings.json`**

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/api-policy.sh"
          }
        ]
      }
    ]
  }
}
```

**Example: `.claude/hooks/api-policy.sh`**

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

### Parallel sessions and subagents

- **Parallel sessions:** A parallel session is a full agent instance. Each one works on its own task in its own Git worktree, so sessions do not overwrite each other's files. The engineer starts and steers them.
- **Subagents:** Subagents are helpers that an agent sends off to handle parts of a job. They are started from a single session, and each one works in its own separate context. Turn repeated jobs (a code simplifier, a verifier, a codebase researcher) into subagents defined in `.claude/agents/`, and check them into Git so the whole team shares them.

**How to run it:**
1. Split the work into tasks that touch different files. Tasks that share files run in one session, one after another.
2. Start with two or three sessions. Add more only while you can still review the output properly.

The engineer's role shifts from writing every line to **orchestrating** sessions and helpers, and eventually to building and monitoring loops.

> **Watch out: higher token use.** Each subagent starts fresh and does not know what the main agent knows. It has to read its own instructions, read the files it needs (even ones the main agent already read) and write its own report. Use subagents for large tasks that split cleanly, not for small ones.

---

## 4. Test

**What changes:** Instead of QA gates at stage boundaries, checks run continuously. The agent verifies its own work before a person sees it, and the configuration that steers the agent is regression-tested like code.

- **Deliverables:** Passing tests, and an eval suite for the agent's configuration
- **Who:** Engineers (feedback loop); platform engineer (eval suite)
- **Where:** The codebase repo and CI
- **Prerequisites:** `AGENTS.md`, plus a test suite and a build that each run with one command

### Give the agent a feedback loop

The agent writes code, runs the checks, fixes what fails and runs them again, until everything passes.

1. Wrap each check in one command, such as `make test`, that fails clearly.
2. List the commands in `AGENTS.md`, with what a healthy result looks like.
3. Give a measurable goal, for example "all tests in `test_status.py` pass".
4. For bug fixes, use test-driven development (red, green, refactor): write a failing test that reproduces the bug, commit it, then have the agent make it pass without changing the test.
5. Protect the tests with a hook that blocks edits to test files during a fix.

**Add to `AGENTS.md`:**

```markdown
## Verifying your work

- Build: make build (must finish with "Build succeeded")
- Test: make test (all green; never skip or delete a failing test)
- Lint: make lint (zero warnings)

Run all three before reporting any task complete, and paste the output.
If a test fails, fix the code, not the test.
```

### Run continuous evals in CI

The feedback loop tests the **code**. Evals test the **agent's setup**.

1. Collect 20 to 50 real tasks from recent work, each with its accepted outcome.
2. Turn each into an eval: the prompt plus the checks that define a good result.
3. Run the suite in CI on a schedule and whenever `AGENTS.md`, skills or hooks change.
4. Review any configuration change that lowers the pass rate before it merges.
5. Add an eval for every production incident, so the same mistake cannot quietly return.

---

## 5. Deploy

**What changes:** Review runs in both directions, and governance is enforced as the agent acts. The agent does everything up to the production gate and nothing past it.

- **Deliverables:** `REVIEW.md` and approval hooks
- **Who:** Tech lead (review policy); platform engineer (hooks)
- **Where:** The repo root and `.claude/settings.json` (or managed settings for rules nobody can switch off)
- **Prerequisites:** `AGENTS.md`, `spec.md`, `plan.md`, and branch protection that requires a code owner's approval
- **Automation:** An AI review runs on every pull request using `REVIEW.md`. Tagging `@claude` on a review comment asks the agent to make the fix.

### AI in the PR review loop

- Every PR gets the same review passes, with findings ranked by severity.
- AI findings do not approve or block a PR. **A code owner always approves.**
- When review flags the same mistake twice, the correction goes into `AGENTS.md`.
- The agent that wrote the code cannot approve it, so separation of duties is preserved.

**Example `REVIEW.md`:**

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

### Hooks as approval gates

Hooks turn required human approvals into checks that run every time, for everyone. For example, a hook can block a production deploy unless a release authorization is present, and explain how to get one. Rules that must never be switched off go in managed settings owned by IT.

Reuse the review bots, skills, agents and hooks your organization already has.

---

## 6. Maintain

**What changes:** The loop closes. A trigger starts the agent with no person in the path, and what it finds re-enters the pipeline as a new `intent.md`.

- **Deliverable:** A new `intent.md` in the triage queue
- **Who:** Service owner or on-call engineer (triage); product owner (product-facing findings)
- **Where:** The intent home and the triage queue
- **Prerequisites:** `intent.md` format, AI PR review, hooks, and a rollback path in CI/CD
- **Automation:** A trigger such as a control-band breach, a ticket, a channel message or a schedule starts the agent.

### How it works

1. Pick one metric with a stable baseline, such as CI test failure rate or the error rate after a deploy.
2. A simple, deterministic script (no AI) watches the metric against its normal range, its control band.
3. Response tiers live in version-controlled config:
   - **1σ (a little unusual):** log it.
   - **2σ (quite unusual):** the agent investigates, read-only.
   - **3σ (very unusual):** the agent may act, but only by opening a PR or running a pre-approved runbook.
4. The agent writes its diagnosis as `intent.md`, which flows through Plan, Design, Build, Test and Deploy like any other change.
5. A person triages each finding: fix now, schedule or dismiss.
6. When a fix ships, add an eval so the problem cannot return unnoticed.

People no longer have to start the work. They triage and review it.

---

## Skills in this repo

| Skill | Stage | What it does |
| :---- | :---- | :---- |
| [`write-intent`](skills/write-intent/SKILL.md) | Plan | Brainstorms like an analyst and writes `intent.md` |
| [`intent-to-spect`](skills/intent-to-spect/SKILL.md) | Design | Turns `intent.md` into `spec.md`, applying brand, security and UX skills and flagging policy conflicts |

## Sources

- [The AI-native SDLC playbook, Claude Academy](https://academy.claude.com/courses/ai-native-sdlc-playbook)
- [Claude Code documentation](https://code.claude.com/docs)
