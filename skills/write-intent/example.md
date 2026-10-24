# Intent: claims status self-service
Author: J. Ortiz (claims operations). Status: draft.

## Problem
Customers phone the contact center to ask where their claim is.
Handlers spend roughly a third of call time on status-only queries.

## Proposed outcome
Customers see claim status, next step and expected date in the portal.
Status-only calls to the contact center drop noticeably.

## Affected users and systems
Customers with an open claim, claims handlers, the portal team,
the claims-core API.

## Constraints
No new PII in the portal session. Existing authentication only.
Out of scope: changing how claims are processed.

## Open questions
Do third-party loss adjusters need access too?
