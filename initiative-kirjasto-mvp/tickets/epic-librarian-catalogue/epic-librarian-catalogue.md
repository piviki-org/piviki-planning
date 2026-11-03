---
type: epic
title: "Librarians keep the catalogue"
parent: initiative-kirjasto-mvp
covers: [R11, R17, R20, R22, R25]
sources: []
companions: [_bmad-output/initiative-kirjasto-mvp/architecture-piviki/architecture-piviki.md]
after: []
refined: true
risk: medium
---

# Librarians keep the catalogue

## Description

On the deployed platform, a librarian adds and edits books and donated copies in an English tool built for translation, with every record owned by the Piviki tenant and its policy values in place. The epic also lays the platform every later epic builds on, and the review, barcode and label tools epic-initial-load needs, as the architecture spine assigns them (AD-5, AD-7, AD-9, AD-12, AD-17, AD-19).

## Outcome

For librarians, a working tool replaces memory for what the library owns, and every later epic builds on its platform, tenant model and catalogue records.

## Requirements

- R1 (parent R11): A librarian can add and edit a book by fast manual entry, without any online metadata existing for it.
- R2 (parent R17, split; Decision 2026-11-03): A librarian can record a copy as donated with a date and no donor, and no donor name is shown to patrons or the public; linking and unlinking a donor belongs to epic-patron-site.
- R3 (parent R20): Loan period, maximum books per patron, renewal rules and limits, retention period, inactivity window, uncollected-reservation window and chat group link are set per tenant, and each can be changed without a code change or blocking launch.
- R4 (parent R22): Librarians use their tool in English, and translating it later needs translation work only.
- R5 (parent R25): Every library-owned record belongs to a tenant and Piviki is the only tenant, so a second tenant could be added without changing the data model.
- R6 (parent R13; spine AD-9): A librarian can review draft records, merge duplicate drafts and publish them in the production review view, which flags draft titles with no copy and images attached to no title, ready for epic-initial-load.
- R7 (spine AD-9; Decision 2026-11-03): A librarian finds a copy by its barcode, typed, entered by a USB scanner, or read by the device camera into the same field, on every screen that acts on a copy.
- R8 (spine AD-9; Decision 2026-11-03): A librarian prints Code 128 barcode labels for copies of published titles on A4 sticker sheets of 20 to 24 labels, starting at any position on a partly used sheet, and the tool records which copies are labelled.

## Constraints

- Milestones are internal targets: catalogue in use with the initial collection (about 300 books) imported by end of November 2026, which needs this epic's review view, API keys and tested restore first (initiative Constraints).
- A two-person part-time team builds and runs this, which is why the cut order exists (initiative Constraints).
- Tenancy stays minimal: tenant ownership of records and per-tenant policy values only (initiative Constraints).
- Email goes through a transactional email service on the library's own domain; a personal Gmail account and Proton are rejected (initiative Constraints).
- Data is minimal, each hosting or SaaS provider has a written data processing agreement, and a breach can be notified within 72 hours (initiative Constraints).
- Donor data, when kept, is donor ID, copy ID and date only, with no free-text names or contact details; only patrons with accounts can be credited, and walk-in donors are anonymous (initiative Constraints).
- A donor's name is never shown publicly without a separate opt-in (initiative Constraints).
- The code and the data belong to the community, not to a subscription service (initiative Constraints).

## Done when

- A librarian adds a book by manual entry, edits it, and records a copy as donated, on the deployed tool, with duplicates refused unless overridden.
- A librarian reviews, merges and publishes draft records in the review view, finds any copy by barcode, and prints labels for published copies.
- Every record belongs to the Piviki tenant, a policy value changes with `policy:set` without a code change, and the tool runs in English on an i18n foundation.
- The platform runs in production behind Cloudflare Tunnel and Access, deployed from CI, with the outside port scan passed, mail sending from the library's domain, and nightly encrypted backups with one restore tested before the bulk import.
- epic-initial-load can run: a librarian account has an API key and the import script has its Access service token.
- Deployed to production and verified end to end.

## Boundaries

- Owns the platform baseline: scaffold, CI, deployment, infrastructure as code, backups, the email adapter and domain, the tenant model and the i18n foundation.
- Owns the collections of spine AD-19: `tenant-policy`, `librarians`, `titles`, `copies`, `media`, `donations`, and `patrons` with no public route.
- Not here: the import script and the bulk import (epic-initial-load), patron-facing pages and sign-up (epic-patron-site), linking and unlinking a donor (epic-patron-site), the outbox and delivery job (epic-reservation-loop), ISBN lookup (epic-loan-care).
- Out: admin screens for tenants or policy values; printer integration beyond printing from the browser.
- Touch point: piviki.fi DNS zone on Cloudflare, imported into OpenTofu with no change to the records serving the coming-soon page; the domain stays registered at OVH outside OpenTofu; owner: epic-librarian-catalogue.
- Must not change: the coming-soon page at piviki.fi stays live.

## References

- parent — _bmad-output/initiative-kirjasto-mvp/initiative-kirjasto-mvp.md, section Requirements
- architecture spine — _bmad-output/initiative-kirjasto-mvp/architecture-piviki/architecture-piviki.md, the AD rules this epic builds to; AD-5, AD-6, AD-7, AD-9, AD-10, AD-11, AD-12, AD-14, AD-16, AD-17, AD-19, AD-20 and Assumptions to Prove bind it most

## Notes

- Decision: 2026-11-03: Email provider is chosen in the email story by a founder, against the spine's criteria: SMTP, the library's own domain, a written data processing agreement, ideally EU-based, open and click tracking off.
- Decision: 2026-11-03: Labels are Code 128 barcodes on A4 sticker sheets of 20 to 24 labels, printed from the browser; the exact sheet product is named in the label story.
- Decision: 2026-11-03: Barcode entry is one field that takes typing or a USB scanner; camera scanning with a barcode library fills the same field and is its own story after labels, cuttable if time runs short.
- Decision: 2026-11-03: Policy values are seeded as placeholders and changed with `policy:set` once stakeholders decide.
- Decision: 2026-11-03: The domain stays registered at OVH; the Cloudflare zone is imported into OpenTofu first, and the first plan must show no changes; a founder creates the scoped Cloudflare API token.
- Decision: 2026-11-03: Labels are built last, so epic-initial-load can start once stories 6 (review view and import access), 7 (tested restore) and 8 (hardening, before real data) are done; its stories pin to those entries, not the whole epic.
- Decision: 2026-11-03: The epic stays whole at 14 stories, because a separate platform epic would gate epic-initial-load just the same and the order already carries the dependencies.
- Decision: 2026-11-03: Story 1 is the tracer bullet: the thinnest path through DNS in OpenTofu, Tunnel, Access, Caddy, core, Postgres and CI, ending with a librarian signed in at admin.piviki.fi in production.
- Decision: 2026-11-03: Backups (story 7) come after the review view and before epic-initial-load, because nothing written before the initial load is irreplaceable; spine AD-17 is amended to say backups run from before the first data that cannot be recreated. Story 7 also removes the test records of stories 3 to 6 before the import.
- Decision: 2026-11-03: No closing end-to-end suite in this epic, because every story tests itself and is checked in production; a Playwright suite on a local Compose stack is proposed when epic-reservation-loop is planned.
- Decision: 2026-11-03: Donor linking moves to epic-patron-site, because no real donor with consent exists before its sign-up and consent box; this epic still creates `donations` with its optional donor field (spine AD-19).
- Decision: 2026-11-03: Production runs on AWS EC2 on the founders' credits until right after the initial load, then moves to OVH VPS-1 by backup restore; the AWS instance then becomes the test environment (spine AD-17). No real patron's data is ever on AWS (AD-8, AD-18).
- Assumption: The seeded placeholder policy values are not used for any real loan before stakeholders set them.
