---
name: write-intent
description: Write up an idea, request, ticket or alert as an intent.md
  proto-spec for product owner review. Use when someone asks to capture,
  write up or log an idea, intent, proto-spec or feature request, says
  "I have an idea", or asks to turn a ticket or incident into an intent.
  Do not use when someone asks Claude to build or fix something directly.
---
# Write an intent.md

An intent.md captures what is wanted, why, and under which constraints,
in the originator's own words. It is the first artifact in the SDLC loop:
once the product owner accepts it, it becomes the input for the
requirements and design spec. It is not a spec: no solution design, no
estimates, no tasks.

An intent can start from a person's idea, a ticket, or an alert or
incident. The same steps apply to all three.

## 1. Read the template and example
Before asking any questions, read `template.md` in this skill's folder.
Its sections tell you what the conversation needs to cover. Read
`example.md` too if you have not written an intent in this session yet.
It shows the length and tone to aim for. Never copy its content.

## 2. Brainstorm until the idea is concrete
Let the originator describe the problem in their own words. No formal
language is required. Then ask the questions an analyst would ask, two
or three at a time, in plain language. Skip anything they have already
answered, and stop once you can fill every section of the template.
Cover:
- Author: their name and team, if not already known.
- Problem: What can't you do today? How often? What does it cost
  (time, money, errors, complaints)?
- Users: Who is affected? Who else would notice the change?
- Success: What does "better" look like? How would we know it worked?
- Systems: Which tools, services or data does this touch, if known?
- Constraints: Any rules on data, privacy, security, compliance,
  deadlines or budget?
- Scope: What is explicitly out of scope?

If the template has sections not listed above, ask about those too.

If the intent starts from a ticket or an alert, read it first and use it
as the source. Ask the person who invoked you only about what the ticket
or alert leaves unclear, and include the ticket or alert ID in the
Problem section so the two records stay linked.

Don't invent answers. If nobody knows, record it under Open questions.
Don't propose technical solutions unless asked.

## 3. Write the file
Follow `template.md` exactly: same headings, same order, same header
line. Replace every placeholder in angle brackets; leave none behind.
Keep it under one page. Use the originator's words where possible, and
include numbers they gave you.

## 4. Let the originator correct it
Show the draft and ask: "Is anything wrong or missing?" Apply their
corrections. The originator must agree before it is committed.

## 5. Commit to the intent home
- File name: `intent/<YYYY-MM>-<kebab-case-name>.md`
- If a GitHub connector or git access is available, create a branch
  named `intent/<kebab-case-name>`, add the file, and open a pull
  request titled "Intent: <short name>". The PR description should be
  a two-sentence summary of the problem and outcome. The product owner
  reviews and corrects it in the PR.
- If no repo access is available, give the originator the final file to
  paste in, and tell them where it goes.

## Rules
- One idea per file. If the conversation covers two ideas, write two.
- Leave Status as draft. The product owner's decision is recorded by the
  PR: merging accepts the intent into design, closing rejects it.
- Don't add a date. Git records the author and timestamp of the commit.
- Never include secrets, passwords or customer personal data. Describe
  the kind of data instead ("customer addresses").
