---
type: epic
title: "Librarians keep the catalogue"
parent: initiative-kirjasto-mvp
covers: [R11, R17, R20, R22, R25]
sources: []
companions: []
after: []
refined: false
risk: medium
---

# Librarians keep the catalogue

## Description

On the deployed platform, a librarian adds and edits books and donated copies in an English tool built for translation, with every record owned by the Piviki tenant and its policy values in place.

## Outcome

For librarians, a working tool replaces memory for what the library owns, and every later epic builds on its platform, tenant model and catalogue records.

## Done when

- A librarian adds a book by manual entry and edits it, on the deployed tool.
- A copy can be recorded as donated, with an optional donor that can later be removed.
- Every record belongs to the Piviki tenant, and a policy value changes without a code change.
- The tool runs in English on an i18n foundation.
- Deployed to production and verified end to end with the epics it builds on.

## Boundaries

- Owns the platform baseline: scaffold, CI, deployment, the tenant model and the i18n foundation.
- Not here: bulk import (epic-initial-load), patron-facing pages (epic-patron-site), ISBN lookup (epic-loan-care).
- Out: admin screens for tenants or policy values.

## References

- parent — _bmad-output/initiative-kirjasto-mvp/initiative-kirjasto-mvp.md, section Requirements
