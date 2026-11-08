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

On the test site, test.piviki.fi, anyone browses and searches in Vietnamese or English, and an adult signs up and is invited to the chat group, and a librarian can link a consenting patron as a copy's donor.

## Outcome

For parents, the collection can be found online in Vietnamese or English, and an adult can open an account and be invited to the chat group.

## Done when

- Anyone browses and searches on test.piviki.fi in Vietnamese or English, with or without diacritics.
- A book page shows availability and its scanned images.
- An adult signs up with the 18+ tick, usage policy acceptance and a separate optional donor consent box, and sees the chat group invitation.
- The pickup address appears nowhere on the site.
- A librarian links a copy to a patron who gave donor consent, and removes the link without deleting the copy or its history.
- While sign-up is closed, the site shows no sign-up, sign-in or Google entry point, from the sign-up state that /api/v1/public reports.
- Deployed to test and verified end to end with the epics it builds on; epic-soft-open makes its first production deploy.

## Boundaries

- Owns touch point: the chat group invitation; join approval is done by a librarian in the chat.
- The site runs only on test.piviki.fi, behind Access, until epic-soft-open; sign-up is limited to the founders' pilot allowlist there.
- No story waits on production: sign-up, Google and donor linking are built and checked on test with made-up or founder patrons. The test environment's own Google OAuth client is set up with sign-up; production's is epic-soft-open's.
- Extends by migration (spine AD-19): `patrons` and the pilot-allowlist gate from epic-librarian-catalogue 1.2, and the `donations` donor field from 1.10.
- Not here: reservations (epic-reservation-loop), the production deploy, policy texts and patron account deletion (epic-soft-open).

## References

- parent — _bmad-output/initiative-kirjasto-mvp/initiative-kirjasto-mvp.md, section Requirements
- architecture spine — _bmad-output/initiative-kirjasto-mvp/architecture-piviki/architecture-piviki.md, the AD rules this epic builds to

## Notes

- Decision: 2026-11-03: Production runs on AWS EC2 on the founders' credits until right after the initial load, then moves to OVH VPS-1 by backup restore; the AWS instance then becomes the test environment (spine AD-17). No real patron's data is ever on AWS (AD-8, AD-18). Superseded 2026-11-05.
- Decision: 2026-11-05: This epic is built and checked on test and no longer waits on epic-initial-load; production and the piviki.fi switch are epic-soft-open's. patronAccountsAllowed is retired; the pilot allowlist gates patron creation while sign-up is closed (spine AD-8).
