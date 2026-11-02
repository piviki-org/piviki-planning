---
type: epic
title: "A book goes from reservation to return"
parent: initiative-kirjasto-mvp
covers: [R4, R5, R7, R8, R14, R15, R18, R24]
sources: []
companions: []
after: []
refined: false
risk: high
---

# A book goes from reservation to return

## Description

A patron reserves or queues, gets the ready email and sees it on their account; a librarian takes it through pickup and return, handles uncollected reservations and adds phone numbers; the overdue block holds.

## Outcome

For parents and librarians, a book moves from reservation to return with the system knowing who has it, who waits and when it is due.

## Done when

- A patron reserves or queues per title, cancels or leaves, and sees loans, requests and due dates on their account.
- A librarian takes a reservation through Ready for pickup, Picked up and Returned, sees loans and queues at a glance, and adds a phone number at first pickup.
- The ready-for-pickup email arrives in the patron's language.
- After the uncollected window a reservation is flagged and is expired only by hand, and those of patrons with no phone number stand out.
- An overdue patron cannot reserve or join a queue.
- Deployed to production and verified end to end with the epics it builds on.

## Boundaries

- Owns touch point: email sending on the library's own domain.
- Never cut (initiative Constraints).
- Not here: renewal and reminder emails (epic-loan-care), lost copies (epic-launch).

## References

- parent — _bmad-output/initiative-kirjasto-mvp/initiative-kirjasto-mvp.md, section Requirements
