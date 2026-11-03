---
type: epic
title: "Parents find books and sign up"
parent: initiative-kirjasto-mvp
covers: [R1, R2, R3, R16, R17]
sources: []
companions: [_bmad-output/initiative-kirjasto-mvp/architecture-piviki/architecture-piviki.md]
after: []
refined: false
risk: medium
---

# Parents find books and sign up

## Description

On the deployed site, not yet at piviki.fi, anyone browses and searches in Vietnamese or English, and an adult signs up and is invited to the chat group, and a librarian can link a consenting patron as a copy's donor.

## Outcome

For parents, the collection can be found online in Vietnamese or English, and an adult can open an account and be invited to the chat group.

## Done when

- Anyone browses and searches on the deployed site in Vietnamese or English, with or without diacritics.
- A book page shows availability and its scanned images.
- An adult signs up with the 18+ tick, usage policy acceptance and a separate optional donor consent box, and sees the chat group invitation.
- The pickup address appears nowhere on the site.
- A librarian links a copy to a patron who gave donor consent, and removes the link without deleting the copy or its history.
- Deployed to production and verified end to end with the epics it builds on.

## Boundaries

- Owns touch point: the chat group invitation; join approval is done by a librarian in the chat.
- The site stays at a non-public address with sign-up closed to real patrons until epic-launch.
- Not here: reservations (epic-reservation-loop), policy texts and account deletion (epic-launch).

## References

- parent — _bmad-output/initiative-kirjasto-mvp/initiative-kirjasto-mvp.md, section Requirements
- architecture spine — _bmad-output/initiative-kirjasto-mvp/architecture-piviki/architecture-piviki.md, the AD rules this epic builds to
