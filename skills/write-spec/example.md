# Spec: claims status self-service
Intent: intent/2026-06-claims-status.md (J. Ortiz, claims operations). Status: draft.
Risk: higher. Customer-facing view of claim data.

## Summary
Signed-in customers will see the status, next step and expected date of
each open claim in the portal, so they stop phoning the contact center
for status-only queries.

## Requirements
- R1. A signed-in customer sees status, next step and expected date for each open claim.
- R2. Status is one of four states: received, in review, approved, paid.
- R3. The response carries those three fields and nothing else.
- R4. What the customer sees is at most 60 seconds old.

## Design
- New endpoint GET /claims/{id}/status in claims-api, behind the gateway JWT.
- claims-api reads the claim from claims-core. The portal never calls claims-core.
- New StatusPanel on the portal claims page. It caches each answer for 60 seconds.
- Mockup: None.

## How each constraint in the intent is met
| Constraint | Met by |
| :---- | :---- |
| No new PII in the portal session | R3. None of the three fields is tagged pii in the schema. |
| Existing authentication only | The gateway JWT. No new login, token or role. |

## Policies applied
| Skill | Version | Effect on this spec |
| :---- | :---- | :---- |
| secure-api-review | a41c9e2 | Gateway JWT on the endpoint; audit event not needed (read-only). |
| brand-voice | 7d03b1f | Wording rules for dates. See concern 2. |
| portal-ux | 1.3.0 | Status panel layout and the four state labels. |

## Open questions from the intent
- Do third-party loss adjusters need access too? Carried forward as concern 1.

## Areas of concern
1. Loss adjusters have no portal account. Giving them access breaks
   "existing authentication only", so they are left out of this spec.
   Options: separate intent for adjuster access, or a new role.
   Owner: security. Status: open.
2. portal-ux says every status shows a date. brand-voice says never to
   state a payment date as a commitment. Both cannot be met as written.
   Options: label it "Estimated date", or show the date only before payment.
   Owner: compliance. Status: open.
3. claims-core allows 50 requests a second. The 60-second cache should keep
   the portal under that, but nobody has load tested it.
   Options: load test before release. Owner: claims-core team. Status: open.

## Out of scope
- Access for third-party loss adjusters.
- Notifications when a status changes.

## Provenance
Prompt: Spec this intent: intent/2026-06-claims-status.md
