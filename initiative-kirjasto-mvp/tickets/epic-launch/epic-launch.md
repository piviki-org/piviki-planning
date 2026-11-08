---
type: epic
title: "Piviki opens"
parent: initiative-kirjasto-mvp
covers: [R19, R23]
sources: []
companions: [_bmad-output/initiative-kirjasto-mvp/architecture-piviki/architecture-piviki.md]
after: []
refined: false
risk: high
---

# Piviki opens

## Description

Retention and inactivity deletion apply to the records, a librarian can delete a patron's account and mark held copies lost, the success signals can be read, and the first real loan is made. The privacy notice, usage policy, patron account deletion and the piviki.fi switch are epic-soft-open's.

## Outcome

For the community, Piviki makes its first real loan with its records kept lawfully.

## Done when

- A librarian can delete a patron's account and mark the held copies lost, with reservations and affected queue places cancelled and those patrons emailed.
- The retention period is applied to loan records, unused accounts are deleted after the inactivity window, and every success signal can be read from the records.
- A first real loan is recorded in production.

## Boundaries

- Not here: the piviki.fi switch, the privacy notice, usage policy and patron account deletion (epic-soft-open).
- Depends on people: stakeholders set the loan values.
- Not here: tenant or donor features beyond the initiative's Boundaries.

## References

- parent — _bmad-output/initiative-kirjasto-mvp/initiative-kirjasto-mvp.md, section Requirements
- architecture spine — _bmad-output/initiative-kirjasto-mvp/architecture-piviki/architecture-piviki.md, the AD rules this epic builds to

## Notes

- Decision: 2026-11-05: R21 and the sign-up side of R19 (privacy notice, usage policy, AD-18 agreements, Art. 9 answer, breach runbook, patron account deletion) move to epic-soft-open; this epic keeps R23, retention and inactivity deletion, the success signals and the first real loan, and waits on sign-up being open.
