---
type: epic
title: "The initial collection is imported"
parent: initiative-kirjasto-mvp
covers: [R13]
sources: []
companions: [_bmad-output/initiative-kirjasto-mvp/architecture-piviki/architecture-piviki.md]
after: []
refined: false
risk: high
---

# The initial collection is imported

## Description

The initial collection of about 300 books is in the live catalogue, created from scans by a script and reviewed by a librarian before bulk import. Production then moves from AWS to OVH, and the AWS instance becomes the test environment (spine AD-17).

## Outcome

For librarians, the initial collection reaches the catalogue without typing each one, in time for the end-of-November milestone.

## Done when

- The script produces records from scanned front covers, back covers and publishing information pages, with Vietnamese diacritics intact.
- A librarian reviews every record before import.
- Every book in the initial collection is in the production catalogue with its images.
- Production has moved to OVH by spine AD-17's seven steps, starting by deleting the check records of epic-librarian-catalogue stories 10 to 13 so only the real catalogue moves: the production Tunnel lists exactly one connector, the outside scan, first-register and hardening checks pass on OVH, and patronAccountsAllowed is true on OVH.
- The AWS instance is wiped and rebuilt as the test environment: test.piviki.fi, test-admin.piviki.fi and test-api.piviki.fi, with Access on the site and admin; its own Tunnel, buckets, OpenTofu state and GitHub Environment; the TST barcode prefix and a TEST mark on labels; mail from testmail.piviki.fi in a separate provider account with the recipient allowlist on; and no backups.

## Boundaries

- Owns the script and the review-and-import path.
- Uses from epic-librarian-catalogue: the review view, the librarian API key and Access service token (1.6), the tested restore (1.7) and the hardening (1.8); its stories pin to those entries.
- Owns the move to OVH and the test environment, planned as the epic's last two stories, the move first.
- Not here: everyday manual entry (epic-librarian-catalogue), ISBN lookup (epic-loan-care).

## References

- parent — _bmad-output/initiative-kirjasto-mvp/initiative-kirjasto-mvp.md, section Requirements
- architecture spine — _bmad-output/initiative-kirjasto-mvp/architecture-piviki/architecture-piviki.md, the AD rules this epic builds to; AD-17 The move and Test environment bind the last two stories

## Notes

- Decision: 2026-11-03: Production runs on AWS EC2 on the founders' credits until right after the initial load, then moves to OVH VPS-1 by backup restore; the AWS instance then becomes the test environment (spine AD-17). No real patron's data is ever on AWS (AD-8, AD-18).
