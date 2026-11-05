---
type: epic
title: "The initial collection is ready to import"
parent: initiative-kirjasto-mvp
covers: [R13]
sources: []
companions: [_bmad-output/initiative-kirjasto-mvp/architecture-piviki/architecture-piviki.md]
after: []
refined: false
risk: high
---

# The initial collection is ready to import

## Description

The import script turns the founder-checked scans of the initial collection, about 300 books, into draft catalogue records, and a librarian reviews them before they are published. Everything here runs on the test environment; the real import into production is epic-soft-open's (spine AD-9, AD-17).

## Outcome

For librarians, the initial collection can reach the catalogue without typing each one, in time for the end-of-November milestone.

## Done when

- The script produces records from scanned front covers, back covers and publishing information pages, with Vietnamese diacritics intact.
- A full trial import of the initial collection has been run on test, reviewed by a librarian and published in test's review view.
- A re-run skips the scans already processed, and test's catalogue and processed-scans file can be reset for another trial.

## Boundaries

- Owns the import script and the review-and-import path, tried on test.
- Uses from epic-librarian-catalogue: the review view, the librarian API key and test's Access service token (1.6); its stories pin to that entry.
- Uses only the founder-checked scans, on test and in production.
- Not here: the real import into production and production's import key and token (epic-soft-open), everyday manual entry (epic-librarian-catalogue), ISBN lookup (epic-loan-care).

## References

- parent — _bmad-output/initiative-kirjasto-mvp/initiative-kirjasto-mvp.md, section Requirements
- architecture spine — _bmad-output/initiative-kirjasto-mvp/architecture-piviki/architecture-piviki.md, the AD rules this epic builds to; AD-9, AD-16, AD-17 Test environment and AD-20 bind it most

## Notes

- Decision: 2026-11-03: Production runs on AWS EC2 on the founders' credits until right after the initial load, then moves to OVH VPS-1 by backup restore; the AWS instance then becomes the test environment (spine AD-17). No real patron's data is ever on AWS (AD-8, AD-18). Superseded 2026-11-05.
- Decision: 2026-11-05: There is no move: AWS is the test environment from its first bootstrap, and production starts empty on OVH in epic-soft-open. This epic no longer owns the move or the test environment.
- Decision: 2026-11-05: This epic builds the import script and runs trial imports on test; the real import is an epic-soft-open story that waits on this epic's script and a reviewed trial import.
- Decision: 2026-11-05: The initial collection's scans have been checked by the founders against the spine's image checklist (no handwritten names, no backgrounds) and are safe to use on test, including in test's public media bucket.
