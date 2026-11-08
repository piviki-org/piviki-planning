---
type: epic
title: "Piviki soft opens"
parent: initiative-kirjasto-mvp
covers: [R13, R19, R21]
sources: []
companions: [_bmad-output/initiative-kirjasto-mvp/architecture-piviki/architecture-piviki.md]
after: []
refined: false
risk: high
---

# Piviki soft opens

## Description

Production starts on OVH, the initial collection is imported into it and published, and piviki.fi then serves the patron website in place of the coming-soon page, where anyone browses and, once the privacy notice, usage policy and processor agreements exist, signs up. Until then the site is browse-only with a founder pilot (spine AD-8, AD-17 Production start and Cutover).

## Outcome

For parents, the collection can be found at piviki.fi, and for librarians the real catalogue is in use in production, ahead of the first loan.

## Done when

- Production runs on one OVH VPS-1, not public before the switch, with the outside scan, hardening, first-register and noindex checks passed, a nightly dump and its heartbeat working, a restore drill passed and mail sending from piviki.fi.
- The initial collection is imported into production, reviewed and published, with no draft from the run left and production's import key and token revoked.
- piviki.fi serves the patron website in place of the coming-soon page, with a tested rollback.
- Sign-up is open to anyone once the privacy notice and usage policy are published, the AD-18 agreements are on file and the Art. 9 question is answered; until then the site shows no sign-up and only the founders' pilot allowlist can sign up.
- A patron can read the privacy notice and delete their account, and pilot accounts are removed by that deletion before sign-up opens.
- Deployed to production and verified end to end with the epics it builds on.

## Boundaries

- Owns touch point: the piviki.fi coming-soon page, replaced by the patron website; Facebook and Instagram links stay outside the system.
- Owns the production start, the real import, the first production patron-site deploy, production's Google OAuth client, the breach runbook and the sign-up checklist.
- Its first two stories wait only on epic-librarian-catalogue and epic-initial-load, so production and the real import can start before epic-patron-site is done.
- Depends on people: a founder runs the production start; the other founder writes the usage policy; the founders write or commission the privacy notice and answer the Art. 9 question.
- Not here: retention and inactivity deletion, a librarian deleting an account and lost copies (epic-launch); refusing deletion while a patron holds a book (epic-reservation-loop, once loans exist).

## References

- parent — _bmad-output/initiative-kirjasto-mvp/initiative-kirjasto-mvp.md, section Requirements
- architecture spine — _bmad-output/initiative-kirjasto-mvp/architecture-piviki/architecture-piviki.md, the AD rules this epic builds to; AD-8, AD-9, AD-14, AD-17 Production start, Backups and Cutover, and AD-18 bind it most

## Notes

- Decision: 2026-11-05: The epic is created between epic-patron-site and epic-reservation-loop, because the founders want no public production until the patron site is ready, opened as a soft open where patrons browse and sign up.
- Decision: 2026-11-05: Its first two stories are planned now, the production start and the real import; the stories after them, which wait on epic-patron-site, are planned when the epic is refined.
- Decision: 2026-11-05: The production host is OVH VPS-1, started by a founder when the founders choose and always before the real import.
- Decision: 2026-11-05: Before sign-up opens, pilot accounts are removed through the normal account deletion, which writes erasure records and is refused while a pilot holds a book; no hard-delete reset (spine AD-8).
- Waits on epic-patron-site because: the patron-site deploy, the piviki.fi switch, production's Google client and the sign-up checklist need the finished patron site; those stories are not planned yet, so no entry pins it.
- Open question: Who writes the privacy notice, and where are it and the usage policy published? The other founder writes the usage policy; founders decide before sign-up opens.
- Open question: Does a Vietnamese community library's patron list count as data revealing ethnic origin under GDPR Art. 9? Founders to check tietosuoja.fi before sign-up opens.
